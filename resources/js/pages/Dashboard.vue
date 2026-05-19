<script setup>
import { ref, computed, onMounted, getCurrentInstance } from 'vue';
import Head from '@/pages/layout/Head.vue';
import DynamicHtmlRenderer from '@/components/DynamicHtmlRenderer.vue';
import { Icon, Button, EmptyStateMenu, EmptyStateItem, DocsCallout } from '@ui';
import { SortableList } from '@/components/sortable/Sortable.js';
import WidgetEditOverlay from '@/components/dashboard/WidgetEditOverlay.vue';
import WidgetPicker from '@/components/dashboard/WidgetPicker.vue';
import WidgetConfigStack from '@/components/dashboard/WidgetConfigStack.vue';
import useArchitecturalBackground from '@/pages/layout/architectural-background.js';
import { router } from '@inertiajs/vue3';
import { clone } from '@/bootstrap/globals.js';

const props = defineProps({
    widgets: Array,
    widgetConfigs: { type: Array, default: () => [] },
    canEditWidgets: { type: Boolean, default: false },
    widgetMetaUrl: String,
    widgetUpdateUrl: String,
    pro: Boolean,
    blueprintsUrl: String,
    collectionsCreateUrl: String,
    navigationCreateUrl: String,
});

const { $axios, $toast } = getCurrentInstance().appContext.config.globalProperties;

const editing = ref(false);
const draftItems = ref([]);
const availableWidgets = ref(null);
const loadingMeta = ref(false);
const configuringIndex = ref(null);
const saving = ref(false);

const widgetsMetaByHandle = computed(() => {
    if (!availableWidgets.value) return {};
    return Object.fromEntries(availableWidgets.value.map((w) => [w.handle, w]));
});

const configuringWidget = computed(() => {
    if (configuringIndex.value === null) return null;
    return draftItems.value[configuringIndex.value]?.config || null;
});

const configuringMeta = computed(() => {
    return configuringWidget.value ? widgetsMetaByHandle.value[configuringWidget.value.type] : null;
});

const unifiedWidgets = computed(() => {
    if (editing.value) {
        return draftItems.value.map((item) => ({ config: item.config, display: item.display }));
    }
    return props.widgets.map((widget) => ({ config: null, display: widget }));
});

onMounted(() => {
    if (!props.widgets.length && !editing.value) useArchitecturalBackground();
});

function tailwindWidthClass(width) {
    const sizes = {
        sm: 'w-full @2xl:w-1/2 @4xl:w-1/3 @7xl:w-1/4',
        md: 'w-full @2xl:w-1/2 @4xl:w-1/2 @7xl:w-1/3',
        lg: 'w-full @2xl:w-full @4xl:w-2/3 @7xl:w-3/4',
        full: 'w-full',
    };
    const legacyMap = { 25: 'sm', 33: 'sm', 50: 'md', 66: 'md', 75: 'lg', 100: 'full' };
    const size = typeof width === 'number' ? (legacyMap[width] ?? 'full') : width;
    return sizes[size] ?? sizes.md;
}

function classes(source) {
    return `${source?.classes ?? ''} ${tailwindWidthClass(source?.width)}`;
}

function ensureMetaLoaded() {
    if (availableWidgets.value || loadingMeta.value) return;
    loadingMeta.value = true;
    $axios.get(props.widgetMetaUrl)
        .then((response) => { availableWidgets.value = response.data; })
        .catch(() => $toast.error(__('Could not load widgets.')))
        .finally(() => { loadingMeta.value = false; });
}

function startEditing() {
    draftItems.value = props.widgetConfigs.map((config, i) => ({
        config: clone(config),
        display: props.widgets[i] || null,
    }));
    editing.value = true;
    ensureMetaLoaded();
}

function cancelEditing() {
    editing.value = false;
    draftItems.value = [];
}

function widgetPicked(widget) {
    const newConfig = { type: widget.handle, ...(widget.defaults || {}) };
    draftItems.value.push({ config: newConfig, display: null });
    configuringIndex.value = draftItems.value.length - 1;
}

function configureWidget(index) {
    configuringIndex.value = index;
}

function widgetConfigSaved(updated) {
    if (configuringIndex.value === null) return;
    const existing = draftItems.value[configuringIndex.value];
    draftItems.value.splice(configuringIndex.value, 1, { ...existing, config: updated });
    configuringIndex.value = null;
}

function removeWidget(index) {
    if (configuringIndex.value === index) {
        configuringIndex.value = null;
    } else if (configuringIndex.value !== null && configuringIndex.value > index) {
        configuringIndex.value--;
    }
    draftItems.value.splice(index, 1);
}

function updateWidth(index, width) {
    const item = draftItems.value[index];
    draftItems.value.splice(index, 1, { ...item, config: { ...item.config, width } });
}

function onSort(sortedItems) {
    if (!editing.value) return;
    draftItems.value = sortedItems.map((item) => ({ config: item.config, display: item.display }));
}

function save() {
    saving.value = true;
    $axios.patch(props.widgetUpdateUrl, { widgets: draftItems.value.map((i) => i.config) })
        .then(() => {
            editing.value = false;
            router.reload();
        })
        .catch(() => $toast.error(__('Something went wrong')))
        .finally(() => { saving.value = false; });
}
</script>

<template>
    <Head :title="__('Dashboard')" />

    <template v-if="editing || widgets.length">
        <ui-header :title="__('Dashboard')" icon="dashboard">
            <template v-if="editing">
                <WidgetPicker :widgets="availableWidgets || []" @picked="widgetPicked" />
                <Button :text="__('Cancel')" @click="cancelEditing" />
                <Button :text="__('Save')" variant="primary" :disabled="saving" @click="save" />
            </template>
            <Button v-else-if="canEditWidgets" :text="__('Edit')" icon="edit" @click="startEditing" />
        </ui-header>

        <SortableList
            :model-value="unifiedWidgets"
            item-class="dashboard-widget-sortable"
            handle-class="dashboard-widget-handle"
            :disabled="!editing"
            :animate="false"
            :constrain-dimensions="true"
            :distance="5"
            @update:model-value="onSort"
        >
            <div class="widgets @container/widgets flex flex-wrap gap-y-6 -mx-2 sm:-mx-3">
                <div
                    v-for="(item, index) in unifiedWidgets"
                    :key="index"
                    class="dashboard-widget-sortable px-3"
                    :class="[classes(item.config ?? item.display), { 'starting-style-transition': !editing }]"
                >
                    <div class="relative">
                        <WidgetEditOverlay
                            v-if="editing && item.config"
                            :config="item.config"
                            :meta="widgetsMetaByHandle[item.config.type]"
                            @configure="configureWidget(index)"
                            @remove="removeWidget(index)"
                            @update:width="updateWidth(index, $event)"
                        />
                        <component v-if="item.display?.component" :is="item.display.component.name" v-bind="item.display.component.props" />
                        <DynamicHtmlRenderer v-else-if="item.display?.html" :html="item.display.html" />
                        <div v-else-if="editing" class="rounded-lg border-2 border-dashed border-gray-300 dark:border-gray-600 bg-gray-50 dark:bg-gray-800/50 p-8 flex flex-col items-center justify-center gap-2 text-gray-500 dark:text-gray-400 min-h-32">
                            <Icon :name="widgetsMetaByHandle[item.config?.type]?.icon ?? 'code-block'" class="size-8 opacity-50" />
                            <span class="text-sm">{{ widgetsMetaByHandle[item.config?.type]?.title ?? item.config?.type }}</span>
                        </div>
                    </div>
                </div>
                <div v-if="editing && !draftItems.length" class="w-full text-center text-gray-500 py-12 border border-dashed rounded-lg dark:border-gray-700">
                    {{ __('No widgets yet. Click "Add Widget" to get started.') }}
                </div>
            </div>
        </SortableList>

        <WidgetConfigStack
            v-if="configuringWidget"
            :config="configuringWidget"
            :meta="configuringMeta"
            @closed="configuringIndex = null"
            @saved="widgetConfigSaved"
        />
    </template>

    <template v-else>
        <header class="py-8 pt-16 text-center">
            <h1 class="text-[25px] font-medium antialiased flex justify-center items-center gap-2 sm:gap-3">
                <Icon name="dashboard" class="size-5 text-gray-500" />
                {{ __('Dashboard') }}
            </h1>
            <div v-if="canEditWidgets" class="mt-4">
                <Button :text="__('Edit Dashboard')" icon="edit" @click="startEditing" />
            </div>
        </header>

        <EmptyStateMenu
            :heading="__('statamic::messages.getting_started_widget_header')"
            :subheading="__('statamic::messages.getting_started_widget_intro')"
        >
            <EmptyStateItem
                href="https://statamic.dev"
                icon="docs"
                :heading="__('Read the Documentation')"
                :description="__('statamic::messages.getting_started_widget_docs')"
            />
            <EmptyStateItem
                v-if="!pro"
                href="https://statamic.dev/licensing"
                icon="pro-ribbon"
                :heading="__('Enable Pro Mode')"
                :description="__('statamic::messages.getting_started_widget_pro')"
            />
            <EmptyStateItem
                :href="blueprintsUrl"
                icon="blueprints"
                :heading="__('Create a Blueprint')"
                :description="__('statamic::messages.blueprints_intro')"
            />
            <EmptyStateItem
                :href="collectionsCreateUrl"
                icon="collections"
                :heading="__('Create a Collection')"
                :description="__('statamic::messages.getting_started_widget_collections')"
            />
            <EmptyStateItem
                :href="navigationCreateUrl"
                icon="navigation"
                :heading="__('Create a Navigation')"
                :description="__('statamic::messages.getting_started_widget_navigation')"
            />
        </EmptyStateMenu>
    </template>

    <DocsCallout :topic="__('Widgets')" url="widgets" />
</template>
