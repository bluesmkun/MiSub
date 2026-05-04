<script setup>
defineProps({
  textSizeClass: {
    type: String,
    default: 'text-lg'
  },
  iconSize: {
    type: Number,
    default: 28
  }
});
</script>

<template>
  <router-link to="/" class="nav-brand-wrap group">
    <div class="nav-brand-badge nav-brand-badge-sm relative" aria-hidden="true">
      <div class="absolute inset-0 rounded-full bg-gradient-to-br from-primary-400/20 to-purple-500/15 dark:from-primary-500/20 dark:to-purple-600/15 blur-md opacity-0 group-hover:opacity-100 transition-opacity duration-500"></div>
      <img :width="iconSize" :height="iconSize" src="/logo.svg" alt="MiSub" class="drop-shadow-sm relative z-10" />
    </div>
    <span class="nav-brand-text" :class="textSizeClass">MiSub</span>
  </router-link>
</template>
