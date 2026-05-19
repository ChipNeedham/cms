<script setup>
import { ref, computed, getCurrentInstance } from 'vue';
import { Stack, Button, PublishContainer, PublishFieldsProvider, PublishFields, Icon } from '@/components/ui';

const props = defineProps({
    config: { type: Object, required: true },
    meta: { type: Object, default: null },
});

const emit = defineEmits(['closed', 'saved']);

const blueprint = computed(() => props.meta?.blueprint);
const title = computed(() => props.meta?.title ?? props.config.type);

function initialValues() {
    const defaults = props.meta?.defaults ?? {};
    const { type, ...rest } = props.config;
    return { ...defaults, ...rest };
}

const values = ref(initialValues());
const fieldMeta = ref(props.meta?.meta || {});
const errors = ref({});
const name = `dashboard-widget-${props.config.type}-${Math.random().toString(36).slice(2, 8)}`;

function save() {
    emit('saved', { type: props.config.type, ...values.value });
}
</script>

<template>
    <Stack
        size="narrow"
        open
        inset
        :title="title"
        @update:open="emit('closed')"
    >
        <div class="flex flex-col h-full">
            <div v-if="!blueprint" class="flex-1 flex items-center justify-center">
                <Icon name="loading" />
            </div>
            <template v-else>
                <div class="flex-1 overflow-auto p-4">
                    <PublishContainer
                        :name="name"
                        :blueprint="blueprint"
                        :meta="fieldMeta"
                        :errors="errors"
                        v-model="values"
                    >
                        <PublishFieldsProvider :fields="blueprint.tabs[0].sections[0].fields">
                            <PublishFields />
                        </PublishFieldsProvider>
                    </PublishContainer>
                </div>
                <div class="border-t bg-gray-200 dark:bg-gray-700 dark:border-gray-900 p-4 flex justify-end gap-2">
                    <Button variant="ghost" :text="__('Cancel')" @click="emit('closed')" />
                    <Button variant="primary" :text="__('Apply')" @click="save" />
                </div>
            </template>
        </div>
    </Stack>
</template>
