<script setup lang="ts">
const colorMode = useColorMode();

const isDark = computed(() => colorMode.value === "dark");

const toggle = () => {
  colorMode.preference = isDark.value ? "light" : "dark";
};
</script>

<template>
  <button class="theme-toggle" :class="{ light: !isDark }" @click="toggle">
    <Transition name="icon" mode="out-in">
      <span v-if="isDark" key="moon" class="icon">🌙</span>
      <span v-else key="sun" class="icon">☀️</span>
    </Transition>
    <span class="toggle-label">{{ isDark ? "Oscuro" : "Claro" }}</span>
  </button>
</template>

<style scoped>
.overlay {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;

  /* Fondo con partículas simuladas via radial-gradients */
  background:
    radial-gradient(
      ellipse at 20% 50%,
      rgba(0, 255, 238, 0.04) 0%,
      transparent 50%
    ),
    radial-gradient(
      ellipse at 80% 20%,
      rgba(124, 58, 237, 0.06) 0%,
      transparent 50%
    ),
    radial-gradient(
      ellipse at 50% 80%,
      rgba(0, 100, 255, 0.05) 0%,
      transparent 50%
    ),
    rgba(3, 4, 15, 0.88);
  backdrop-filter: blur(12px);
}

/* Modo claro — amanecer espacial */
:global(.light) .overlay {
  background:
    radial-gradient(
      ellipse at 20% 50%,
      rgba(255, 180, 50, 0.06) 0%,
      transparent 50%
    ),
    radial-gradient(
      ellipse at 80% 20%,
      rgba(255, 100, 30, 0.07) 0%,
      transparent 50%
    ),
    radial-gradient(
      ellipse at 50% 80%,
      rgba(50, 100, 255, 0.05) 0%,
      transparent 50%
    ),
    rgba(8, 18, 45, 0.88);
  backdrop-filter: blur(12px);
}

.modal {
  width: 100%;
  max-width: 560px;
  max-height: 90vh;
  overflow-y: auto;
  padding: 0;

  /* Modo oscuro */
  background: rgba(3, 6, 25, 0.75);
  border: 1px solid rgba(0, 255, 238, 0.18);
  border-radius: 20px;
  box-shadow:
    0 0 0 1px rgba(0, 255, 238, 0.04) inset,
    0 8px 60px rgba(0, 0, 0, 0.7),
    0 0 60px rgba(0, 200, 255, 0.06);
  backdrop-filter: blur(24px);
}

/* Modal modo claro */
:global(.light) .modal {
  background: rgba(8, 18, 50, 0.78);
  border-color: rgba(255, 180, 50, 0.22);
  box-shadow:
    0 0 0 1px rgba(255, 180, 50, 0.04) inset,
    0 8px 60px rgba(0, 0, 0, 0.6),
    0 0 60px rgba(255, 120, 0, 0.08);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.2rem 1.5rem;
  border-bottom: 1px solid rgba(0, 255, 238, 0.1);
  background: rgba(0, 0, 0, 0.25);
  border-radius: 20px 20px 0 0;
}

:global(.light) .modal-header {
  border-bottom-color: rgba(255, 180, 50, 0.12);
}

/* Etiquetas de campos en modo claro — dorado */
:global(.light) .form-label {
  color: #ffd700;
}
:global(.light) .modal-icon {
  color: #ffd700;
}
:global(.light) .modal-title {
  color: #e8f4ff;
}

/* Inputs en modo claro */
:global(.light) .form-input {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(255, 180, 50, 0.2);
  color: #e8f4ff;
}
:global(.light) .form-input:focus {
  border-color: rgba(255, 180, 50, 0.5);
  box-shadow: 0 0 0 3px rgba(255, 180, 50, 0.06);
}

/* Botón submit en modo claro — dorado */
:global(.light) .submit-btn {
  background: #ffd700;
  color: #0a0a1a;
}
:global(.light) .submit-btn:hover:not(:disabled) {
  background: #ffe44d;
}

/* Botón cancelar en modo claro */
:global(.light) .cancel-btn {
  border-color: rgba(255, 180, 50, 0.25);
  color: rgba(200, 220, 255, 0.7);
}
:global(.light) .cancel-btn:hover {
  border-color: rgba(255, 180, 50, 0.5);
  color: #ffd700;
}

/* Success state */
:global(.light) .success-icon {
  border-color: #ffd700;
  color: #ffd700;
  background: rgba(255, 215, 0, 0.1);
}

.theme-toggle {
  display: flex;
  align-items: center;
  gap: 6px;
  background: rgba(0, 255, 238, 0.06);
  border: 1px solid rgba(0, 255, 238, 0.2);
  border-radius: 20px;
  padding: 5px 12px 5px 8px;
  cursor: pointer;
  transition: all 0.25s ease;
  color: #00ffee;
  font-family: "Fira Code", monospace;
  font-size: 0.72rem;
  letter-spacing: 0.08em;
}

.theme-toggle:hover {
  background: rgba(0, 255, 238, 0.12);
  border-color: rgba(0, 255, 238, 0.4);
  box-shadow: 0 0 12px rgba(0, 255, 238, 0.15);
}

.theme-toggle.light {
  background: rgba(100, 0, 220, 0.08);
  border-color: rgba(120, 60, 220, 0.3);
  color: #5b21b6;
}

.theme-toggle.light:hover {
  background: rgba(100, 0, 220, 0.15);
  box-shadow: 0 0 12px rgba(100, 0, 220, 0.2);
}

.icon {
  font-size: 0.95rem;
}
.toggle-label {
  line-height: 1;
}

/* Transición del icono */
.icon-enter-active,
.icon-leave-active {
  transition: all 0.2s ease;
}
.icon-enter-from {
  opacity: 0;
  transform: rotate(-90deg) scale(0.5);
}
.icon-leave-to {
  opacity: 0;
  transform: rotate(90deg) scale(0.5);
}
</style>
