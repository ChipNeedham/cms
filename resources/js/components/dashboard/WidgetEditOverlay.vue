<script setup>
import { Icon } from '@/components/ui';
import { computed } from 'vue';

const props = defineProps({
    config: { type: Object, required: true },
    meta: { type: Object, default: null },
});

const emit = defineEmits(['configure', 'remove', 'update:width']);

const widthOptions = ['sm', 'md', 'lg', 'full'];
const currentWidth = computed(() => props.config.width ?? 'md');
</script>

<template>
    <div class="absolute inset-0 z-10 rounded-lg ring-2 ring-inset ring-blue-500/40">
        <div class="absolute inset-x-0 top-0 flex items-center gap-1 px-2 py-2 rounded-t-lg bg-gray-900/80 backdrop-blur-sm">
            <Icon name="handles" class="dashboard-widget-handle cursor-grab text-white size-4 shrink-0 drop-shadow-sm" />
            <Icon :name="meta?.icon ?? 'code-block'" class="size-4 text-white/70 shrink-0" />
            <span class="text-sm text-white font-medium truncate min-w-0">{{ meta?.title ?? config.type }}</span>

            <div class="flex-1 min-w-2" />

            <div class="flex items-center text-xs font-medium rounded overflow-hidden border border-white/20">
                <button
                    v-for="opt in widthOptions"
                    :key="opt"
                    type="button"
                    class="px-2 py-1 text-white/70 hover:text-white hover:bg-white/10 transition-colors"
                    :class="{ 'bg-white/25 !text-white': currentWidth === opt }"
                    @click="emit('update:width', opt)"
                >{{ opt.charAt(0).toUpperCase() + opt.slice(1) }}</button>
            </div>

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
