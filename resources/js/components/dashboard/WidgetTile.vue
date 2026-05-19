<script setup>
import { Button, Icon, Dropdown, DropdownMenu, DropdownItem } from '@/components/ui';
import { computed } from 'vue';

const props = defineProps({
    config: { type: Object, required: true },
    meta: { type: Object, default: null },
});

const emit = defineEmits(['configure', 'remove', 'update:width']);

const title = computed(() => props.meta?.title ?? props.config.type);
const icon = computed(() => props.meta?.icon ?? 'generic-widget');
const description = computed(() => props.meta?.description);
const widthOptions = ['sm', 'md', 'lg', 'full'];
const currentWidth = computed(() => props.config.width ?? 'md');
</script>

<template>
    <div class="dashboard-widget-tile relative flex flex-col gap-2 rounded-lg border border-dashed border-gray-400 bg-gray-100 dark:bg-gray-800 dark:border-gray-700 p-4">
        <div class="flex items-start gap-3">
            <Icon name="handles" class="dashboard-widget-handle cursor-grab text-gray-500 size-4 shrink-0 mt-1" />
            <Icon :name="icon" class="size-5 text-gray-500 shrink-0 mt-1" />
            <div class="flex-1 min-w-0 cursor-pointer" @click="emit('configure')">
                <div class="font-medium truncate">{{ title }}</div>
                <div v-if="description" class="text-xs text-gray-600 dark:text-gray-400 truncate">{{ description }}</div>
            </div>

            <Dropdown placement="bottom-end">
                <template #trigger>
                    <Button size="sm" icon="more-horizontal" variant="ghost" :aria-label="__('Width')" />
                </template>
                <DropdownMenu>
                    <DropdownItem
                        v-for="opt in widthOptions"
                        :key="opt"
                        :text="__(opt.charAt(0).toUpperCase() + opt.slice(1))"
                        :icon="currentWidth === opt ? 'check' : undefined"
                        @click="emit('update:width', opt)"
                    />
                </DropdownMenu>
            </Dropdown>

            <Button size="sm" icon="settings" variant="ghost" :aria-label="__('Configure')" @click="emit('configure')" />
            <Button size="sm" icon="trash" variant="ghost" :aria-label="__('Remove')" @click="emit('remove')" />
        </div>
    </div>
</template>
