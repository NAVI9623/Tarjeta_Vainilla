<script setup lang="ts">
import Background from "~/components/Background.vue";
</script>

<template>
  <div class="container">
    <AppHeader />
    <ColorModeButton />
    <AppFooter />
    <ClientOnly>
      <Background />
    </ClientOnly>
    <slot></slot>
  </div>
</template>
