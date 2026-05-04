<script setup>
import { computed } from 'vue';
import { useUIStore } from '../../stores/ui.js';
import { useSessionStore } from '../../stores/session.js';
import BrandLogo from './BrandLogo.vue';
import NavActionGroup from './NavActionGroup.vue';

const uiStore = useUIStore();
const sessionStore = useSessionStore();

const shouldHideLoginButton = computed(() => {
  if (sessionStore.sessionState === 'loading') {
    return true;
  }

  const rawPath = sessionStore.publicConfig?.customLoginPath;
  if (!rawPath || typeof rawPath !== 'string') {
    return false;
  }

  const normalizedPath = rawPath.trim().replace(/^\/+/, '');
  if (!normalizedPath || normalizedPath === 'login') {
    return false;
  }

  return true;
});

const props = defineProps({
  isLoggedIn: Boolean,
  hideBranding: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['logout']);
</script>

<template>
  <header
    class="app-nav-bar ios-fixed-header bg-gradient-to-b from-white/95 via-white/90 to-white/85 dark:from-[#030712]/95 dark:via-[#030712]/90 dark:to-[#030712]/85 backdrop-blur-3xl sticky top-0 z-50 border-b border-gray-200/15 dark:border-white/5 transition-all duration-300"
  >
    <div class="ios-status-overlay"></div>

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="safe-top-inset">
        <div class="flex justify-between items-center h-16 md:h-[76px]">
          <BrandLogo v-if="!hideBranding" text-size-class="text-lg" :icon-size="32" />
          <router-link v-else to="/" class="text-sm font-semibold tracking-[0.24em] uppercase text-gray-400 dark:text-gray-500 hover:text-gray-600 dark:hover:text-gray-300 transition-colors">Home</router-link>

          <NavActionGroup
            :is-logged-in="isLoggedIn"
            :show-explore="false"
            :show-settings="true"
            :show-login-button="!shouldHideLoginButton"
            :hide-external-repo="hideBranding"
            :with-focus-ring="true"
            rounded-class="rounded-xl"
            @open-settings="uiStore.show()"
            @toggle-layout="uiStore.toggleLayout()"
            @logout="emit('logout')"
          />
        </div>
      </div>
    </div>
  </header>
</template>
