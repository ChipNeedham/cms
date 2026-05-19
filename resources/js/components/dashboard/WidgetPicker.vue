<script setup>
import { Stack, Icon, Button } from '@/components/ui';

const props = defineProps({
    widgets: { type: Array, required: true },
});

const emit = defineEmits(['closed', 'picked']);
</script>

<template>
    <Stack
        size="narrow"
        open
        inset
        :title="__('Add Widget')"
        @update:open="emit('closed')"
    >
        <div class="p-4">
            <div v-if="!widgets.length" class="text-center text-gray-500 py-8">
                {{ __('No widgets available.') }}
            </div>
            <div v-else class="grid grid-cols-1 gap-1.5">
                <button
                    v-for="widget in widgets"
                    :key="widget.handle"
                    type="button"
                    class="flex items-center gap-2 w-full px-3 py-2.5 group bg-white dark:bg-gray-850 shadow-ui-sm rounded-xl border border-gray-200 dark:border-x-0 dark:border-b-0 dark:border-gray-700 cursor-pointer"
                    @click="emit('picked', widget)"
                >
                    <Icon :name="widget.icon" class="size-5 shrink-0 text-gray-500 group-hover:text-gray-900 dark:text-gray-400 dark:group-hover:text-gray-100" />
                    <span class="text-sm text-gray-700 dark:text-gray-300 group-hover:text-gray-900 dark:group-hover:text-gray-100">{{ widget.title }}</span>
                </button>
            </div>
        </div>
    </Stack>
</template>
