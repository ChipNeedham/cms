<script>
import { Stack, Button, PublishContainer, PublishTabs, Icon } from '@/components/ui';

export default {
    components: { Stack, Button, PublishContainer, PublishTabs, Icon },

    emits: ['closed', 'saved'],

    props: {
        config: { type: Object, required: true },
        meta: { type: Object, default: null },
    },

    data() {
        return {
            values: this.initialValues(),
            fieldMeta: this.meta?.meta || {},
            errors: {},
            name: `dashboard-widget-${this.config.type}-${Math.random().toString(36).slice(2, 8)}`,
        };
    },

    computed: {
        blueprint() {
            return this.meta?.blueprint;
        },

        title() {
            return this.meta?.title ?? this.config.type;
        },
    },

    methods: {
        initialValues() {
            const defaults = this.meta?.defaults ?? {};
            const { type, ...rest } = this.config;
            return { ...defaults, ...rest };
        },

        save() {
            const next = { type: this.config.type, ...this.values };
            this.$emit('saved', next);
        },
    },
};
</script>

<template>
    <Stack
        size="narrow"
        open
        inset
        :title="title"
        @update:open="$emit('closed')"
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
                        <PublishTabs />
                    </PublishContainer>
                </div>
                <div class="border-t bg-gray-200 dark:bg-gray-700 dark:border-gray-900 p-4 flex justify-end gap-2">
                    <Button variant="ghost" :text="__('Cancel')" @click="$emit('closed')" />
                    <Button variant="primary" :text="__('Apply')" @click="save" />
                </div>
            </template>
        </div>
    </Stack>
</template>
