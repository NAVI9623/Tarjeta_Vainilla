<script setup lang="ts">
const modalOpen = ref(false);
</script>

<template>
  <div class="card-footer">
    <ButtonGitHub />
    <ButtonContact @open="modalOpen = true" />
    <ContactModal :open="modalOpen" @close="modalOpen = false" />
  </div>
</template>

<style scoped>
.card-footer {
  padding: 1.2rem 2rem 1.8rem;
  display: flex;
  justify-content: center;
  gap: 1rem;
  border-top: 1px solid rgba(0, 255, 238, 0.08);
}
</style>
