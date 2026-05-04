<script setup>
import { defineAsyncComponent, computed, watchEffect } from 'vue';
import { useSessionStore } from '../stores/session';
import { storeToRefs } from 'pinia';
import { useRoute, useRouter } from 'vue-router';

const PublicProfilesView = defineAsyncComponent(() => import('./PublicProfilesView.vue'));
const LandingView = defineAsyncComponent(() => import('./LandingView.vue'));
const NotFoundView = defineAsyncComponent(() => import('./NotFound.vue'));

const sessionStore = useSessionStore();
const { sessionState, publicConfig } = storeToRefs(sessionStore);
const route = useRoute();
const router = useRouter();

const isExploreRoute = computed(() => route.path === '/explore');
const isRootPath = computed(() => route.path === '/');

watchEffect(() => {
    if (sessionState.value === 'loggedIn' && !isExploreRoute.value) {
        router.replace('/dashboard');
    }
});

const currentView = computed(() => {
    if (isExploreRoute.value) {
        if (publicConfig.value && !publicConfig.value.enablePublicPage) {
            return NotFoundView;
        }
        return PublicProfilesView;
    }

    if (isRootPath.value) {
        if (sessionState.value === 'loggedIn') {
            return { template: '<div></div>' };
        }
        
        if (sessionState.value === 'loggedOut' && publicConfig.value && !publicConfig.value.enablePublicPage) {
            return NotFoundView;
        }
        
        return LandingView;
    }
    
    return PublicProfilesView;
});
</script>

<template>
    <component :is="currentView" />
</template>
