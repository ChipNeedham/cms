<script>
import Head from '@/pages/layout/Head.vue';
import DynamicHtmlRenderer from '@/components/DynamicHtmlRenderer.vue';
import { Icon, Button, EmptyStateMenu, EmptyStateItem, DocsCallout } from '@ui';
import { SortableList } from '@/components/sortable/Sortable.js';
import WidgetTile from '@/components/dashboard/WidgetTile.vue';
import WidgetPicker from '@/components/dashboard/WidgetPicker.vue';
import WidgetConfigStack from '@/components/dashboard/WidgetConfigStack.vue';
import useArchitecturalBackground from '@/pages/layout/architectural-background.js';
import { router } from '@inertiajs/vue3';
import { clone } from '@/bootstrap/globals.js';

export default {
    components: {
        Head,
        DynamicHtmlRenderer,
        Icon,
        Button,
        EmptyStateMenu,
        EmptyStateItem,
        DocsCallout,
        SortableList,
        WidgetTile,
        WidgetPicker,
        WidgetConfigStack,
    },

    props: {
        widgets: Array,
        widgetConfigs: { type: Array, default: () => [] },
        canEditWidgets: { type: Boolean, default: false },
        widgetMetaUrl: String,
        widgetUpdateUrl: String,
        pro: Boolean,
        blueprintsUrl: String,
        collectionsCreateUrl: String,
        navigationCreateUrl: String,
    },

    data() {
        return {
            editing: false,
            draftWidgets: [],
            availableWidgets: null,
            loadingMeta: false,
            picking: false,
            configuringIndex: null,
            saving: false,
        };
    },

    computed: {
        widgetsMetaByHandle() {
            if (! this.availableWidgets) return {};
            return Object.fromEntries(this.availableWidgets.map((w) => [w.handle, w]));
        },

        configuringWidget() {
            if (this.configuringIndex === null) return null;
            return this.draftWidgets[this.configuringIndex] || null;
        },

        configuringMeta() {
            return this.configuringWidget ? this.widgetsMetaByHandle[this.configuringWidget.type] : null;
        },
    },

    created() {
        if (!this.widgets.length && !this.editing) useArchitecturalBackground();
    },

    methods: {
        classes(widget) {
            return `${widget.classes ?? ''} ${this.tailwindWidthClass(widget.width)}`;
        },

        tailwindWidthClass(width) {
            const sizes = {
                sm: 'w-full @2xl:w-1/2 @4xl:w-1/3 @7xl:w-1/4',
                md: 'w-full @2xl:w-1/2 @4xl:w-1/2 @7xl:w-1/3',
                lg: 'w-full @2xl:w-full @4xl:w-2/3 @7xl:w-3/4',
                full: 'w-full',
            };

            const legacyMap = { 25: 'sm', 33: 'sm', 50: 'md', 66: 'md', 75: 'lg', 100: 'full' };
            const size = typeof width === 'number' ? (legacyMap[width] ?? 'full') : width;
            return sizes[size] ?? sizes.md;
        },

        startEditing() {
            this.draftWidgets = clone(this.widgetConfigs);
            this.editing = true;
            this.ensureMetaLoaded();
        },

        cancelEditing() {
            this.editing = false;
            this.draftWidgets = [];
        },

        ensureMetaLoaded() {
            if (this.availableWidgets || this.loadingMeta) return;
            this.loadingMeta = true;
            this.$axios.get(this.widgetMetaUrl)
                .then((response) => { this.availableWidgets = response.data; })
                .catch(() => this.$toast.error(__('Could not load widgets.')))
                .finally(() => { this.loadingMeta = false; });
        },

        openPicker() {
            this.ensureMetaLoaded();
            this.picking = true;
        },

        widgetPicked(widget) {
            const newConfig = { type: widget.handle, ...(widget.defaults || {}) };
            this.draftWidgets.push(newConfig);
            this.picking = false;
            this.configuringIndex = this.draftWidgets.length - 1;
        },

        configureWidget(index) {
            this.configuringIndex = index;
        },

        widgetConfigSaved(updated) {
            if (this.configuringIndex === null) return;
            this.draftWidgets.splice(this.configuringIndex, 1, updated);
            this.configuringIndex = null;
        },

        removeWidget(index) {
            this.draftWidgets.splice(index, 1);
        },

        updateWidth(index, width) {
            this.draftWidgets[index] = { ...this.draftWidgets[index], width };
        },

        save() {
            this.saving = true;
            this.$axios.patch(this.widgetUpdateUrl, { widgets: this.draftWidgets })
                .then(() => {
                    this.editing = false;
                    router.reload();
                })
                .catch(() => this.$toast.error(__('Something went wrong')))
                .finally(() => { this.saving = false; });
        },

    },
};
</script>

<template>
    <Head :title="__('Dashboard')" />

    <template v-if="editing">
        <ui-header :title="__('Dashboard')" icon="dashboard">
            <Button :text="__('Add Widget')" icon="plus" @click="openPicker" />
            <Button :text="__('Cancel')" @click="cancelEditing" />
            <Button :text="__('Save')" variant="primary" :disabled="saving" @click="save" />
        </ui-header>

        <SortableList
            v-model="draftWidgets"
            item-class="dashboard-widget-sortable"
            handle-class="dashboard-widget-handle"
            :vertical="true"
            :mirror="false"
            :distance="5"
        >
            <div class="flex flex-col gap-3">
                <div
                    v-for="(config, index) in draftWidgets"
                    :key="`${config.type}-${index}`"
                    class="dashboard-widget-sortable"
                >
                    <WidgetTile
                        :config="config"
                        :meta="widgetsMetaByHandle[config.type]"
                        @configure="configureWidget(index)"
                        @remove="removeWidget(index)"
                        @update:width="updateWidth(index, $event)"
                    />
                </div>
                <div v-if="!draftWidgets.length" class="text-center text-gray-500 py-12 border border-dashed rounded-lg dark:border-gray-700">
                    {{ __('No widgets yet. Click "Add Widget" to get started.') }}
                </div>
            </div>
        </SortableList>

        <WidgetPicker
            v-if="picking"
            :widgets="availableWidgets || []"
            @closed="picking = false"
            @picked="widgetPicked"
        />

        <WidgetConfigStack
            v-if="configuringWidget"
            :config="configuringWidget"
            :meta="configuringMeta"
            @closed="configuringIndex = null"
            @saved="widgetConfigSaved"
        />

    </template>

    <template v-else-if="widgets.length">
        <ui-header :title="__('Dashboard')" icon="dashboard">
            <Button v-if="canEditWidgets" :text="__('Edit')" icon="edit" @click="startEditing" />
        </ui-header>

        <div class="widgets @container/widgets flex flex-wrap gap-y-6 -mx-2 sm:-mx-3">
            <div
                v-for="widget in widgets"
                class="px-3 starting-style-transition"
                :class="classes(widget)"
            >
                <component v-if="widget.component" :is="widget.component.name" v-bind="widget.component.props" />
                <DynamicHtmlRenderer v-else :html="widget.html" />
            </div>
        </div>
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
