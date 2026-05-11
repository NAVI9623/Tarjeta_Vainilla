<script setup lang="ts">
import { defineAsyncComponent } from "vue";

defineProps<{ open: boolean }>();
defineEmits<{ close: [] }>();

// ✅ CODE SPLITTING — Three.js del ajedrez en su propio chunk
const ChessGame = defineAsyncComponent({
  loader: () => import("~/components/ChessGame.vue"),
  delay: 200,
  timeout: 15000,
});
</script>

<template>
  <Teleport to="body">
    <Transition name="chess-modal">
      <div v-if="open" class="chess-overlay">
        <!-- Header -->
        <div class="chess-header">
          <div class="chess-title-wrap">
            <span class="chess-icon">♟</span>
            <h2 class="chess-title">Ajedrez Espacial</h2>
            <span class="chess-sub">Three.js · Space Edition</span>
          </div>
          <button class="chess-close" @click="$emit('close')">✕</button>
        </div>

        <!-- Juego -->
        <div class="chess-body">
          <ClientOnly>
            <ChessGame />
            <template #fallback>
              <div class="chess-fallback">
                <div class="chess-spinner"></div>
                <p>Preparando el tablero espacial...</p>
              </div>
            </template>
          </ClientOnly>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.chess-overlay {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: flex;
  flex-direction: column;
  background: rgba(2, 3, 15, 0.97);
  backdrop-filter: blur(16px);
}

.chess-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.5rem;
  border-bottom: 1px solid rgba(0, 255, 238, 0.1);
  background: rgba(0, 0, 0, 0.4);
  flex-shrink: 0;
}

.chess-title-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
}
.chess-icon {
  font-size: 1.4rem;
  color: #7c3aed;
}
.chess-title {
  font-family: "Orbitron", monospace;
  font-size: 1.1rem;
  font-weight: 700;
  color: #fff;
  margin: 0;
}
.chess-sub {
  font-family: "Fira Code", monospace;
  font-size: 0.65rem;
  color: rgba(0, 255, 238, 0.5);
  letter-spacing: 0.1em;
  text-transform: uppercase;
  margin-left: 4px;
}

.chess-close {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: rgba(160, 160, 200, 0.7);
  width: 34px;
  height: 34px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}
.chess-close:hover {
  background: rgba(255, 60, 60, 0.12);
  border-color: rgba(255, 60, 60, 0.35);
  color: #ff5050;
}

.chess-body {
  flex: 1;
  overflow: hidden;
}

.chess-fallback {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  background: #02030f;
}
.chess-fallback p {
  color: #00ffee;
  font-family: "Orbitron", monospace;
  font-size: 0.85rem;
}
.chess-spinner {
  width: 52px;
  height: 52px;
  border: 3px solid rgba(0, 255, 238, 0.15);
  border-top-color: #00ffee;
  border-radius: 50%;
  animation: spin 0.9s linear infinite;
}
@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Transición */
.chess-modal-enter-active,
.chess-modal-leave-active {
  transition: all 0.3s ease;
}
.chess-modal-enter-from,
.chess-modal-leave-to {
  opacity: 0;
  transform: scale(0.97);
}
</style>
