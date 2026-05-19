<script setup>
import { Icon } from '@/components/ui';
import WidthSelector from '@/components/fields/WidthSelector.vue';
import { computed } from 'vue';

const props = defineProps({
    config: { type: Object, required: true },
    meta: { type: Object, default: null },
});

const emit = defineEmits(['configure', 'remove', 'update:width']);

const widthToPercent = { sm: 25, md: 50, lg: 75, full: 100 };
const percentToWidth = { 25: 'sm', 50: 'md', 75: 'lg', 100: 'full' };

const currentWidth = computed(() => widthToPercent[props.config.width ?? 'md']);

function onWidthUpdate(value) {
    emit('update:width', percentToWidth[value]);
}
</script>

<template>
    <div class="absolute inset-0 z-[2] rounded-lg ring-2 ring-inset ring-blue-500/40">
        <div class="absolute inset-x-0 top-0 flex items-center gap-1 px-2 py-2 rounded-t-lg bg-gray-900/80 backdrop-blur-sm">
            <Icon name="handles" class="dashboard-widget-handle cursor-grab text-white size-4 shrink-0 drop-shadow-sm" />
            <Icon :name="meta?.icon ?? 'code-block'" class="size-4 text-white/70 shrink-0" />
            <span class="text-sm text-white font-medium truncate min-w-0">{{ meta?.title ?? config.type }}</span>

            <div class="flex-1 min-w-2" />

            <WidthSelector
                :model-value="currentWidth"
                :initial-widths="[25, 50, 75, 100]"
                @update:model-value="onWidthUpdate"
            />

            <button
                type="button"
                class="p-1.5 rounded text-white hover:bg-white/20 transition-colors"
                :aria-label="__('Configure')"
                @click="emit('configure')"
            >
                <Icon name="configure" class="size-4 drop-shadow-sm" />
            </button>

            <button
                type="button"
                class="p-1.5 rounded text-white hover:bg-red-500/60 transition-colors"
                :aria-label="__('Remove')"
                @click="emit('remove')"
            >
                <Icon name="trash" class="size-4 drop-shadow-sm" />
            </button>
        </div>
    </div>
</template>
