<script setup>
import { computed } from 'vue';
import { useVersionStore } from '../../stores/version';
import packageJson from '../../../package.json';

defineProps({
  hideBranding: {
    type: Boolean,
    default: false
  }
});

const versionStore = useVersionStore();
const currentYear = computed(() => new Date().getFullYear());
const currentVersion = packageJson.version;
</script>

<template>
<footer class="w-full text-center p-8 border-t border-gray-200/30 dark:border-white/5 mt-8 bg-white/20 dark:bg-transparent backdrop-blur-md">
  <div class="flex flex-col sm:flex-row items-center justify-center gap-3 text-xs text-gray-400 dark:text-gray-500">
    <p>
      <template v-if="!hideBranding">
        <span class="font-semibold text-indigo-600/80 dark:text-indigo-400/80">MiSUB</span>
        &copy; {{ currentYear }}. All Rights Reserved.
      </template>
      <template v-else>
        &copy; {{ currentYear }}. All Rights Reserved.
      </template>
    </p>
    <span class="hidden sm:inline text-gray-300 dark:text-gray-700">|</span>
    <button 
      @click="versionStore.openModal"
      class="px-2 py-0.5 rounded-lg bg-gray-100/60 dark:bg-white/5 text-[10px] font-mono hover:text-primary-500 dark:hover:text-primary-400 transition-all cursor-pointer opacity-70 hover:opacity-100 border border-transparent hover:border-primary-500/20 hover:shadow-sm"
    >
      v{{ currentVersion }}
    </button>
  </div>
</footer>
</template>
