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
        :title="__('Add Widget')"
        @update:open="emit('closed')"
    >
        <div class="p-4">
            <div v-if="!widgets.length" class="text-center text-gray-500 py-8">
                {{ __('No widgets available.') }}
            </div>
            <div v-else class="grid grid-cols-1 gap-3">
                <button
                    v-for="widget in widgets"
                    :key="widget.handle"
                    type="button"
                    class="text-start flex items-start gap-3 p-3 rounded-lg border bg-white dark:bg-gray-800 dark:border-gray-700 hover:border-blue-500 hover:shadow"
                    @click="emit('picked', widget)"
                >
                    <Icon :name="widget.icon" class="size-5 text-gray-500 shrink-0 mt-1" />
                    <div class="flex-1 min-w-0">
                        <div class="font-medium">{{ widget.title }}</div>
                        <div v-if="widget.description" class="text-xs text-gray-600 dark:text-gray-400">
                            {{ widget.description }}
                        </div>
                    </div>
                </button>
            </div>
        </div>
    </Stack>
</template>
