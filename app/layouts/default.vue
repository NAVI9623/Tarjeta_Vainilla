<script setup lang="ts">
import Background from "~/components/Background.vue";

const colorMode = useColorMode();
const cardRef = ref<HTMLElement | null>(null);

const onMouseMove = (e: MouseEvent) => {
  const card = cardRef.value;
  if (!card) return;
  const rect = card.getBoundingClientRect();
  const x = (e.clientX - rect.left) / rect.width - 0.5;
  const y = (e.clientY - rect.top) / rect.height - 0.5;
  card.style.transform = `perspective(1000px) rotateY(${x * 8}deg) rotateX(${-y * 8}deg)`;
};

const onMouseLeave = () => {
  const card = cardRef.value;
  if (!card) return;
  card.style.transform = "perspective(1000px) rotateY(0deg) rotateX(0deg)";
};
</script>

<template>
  <ClientOnly>
    <Background />
  </ClientOnly>

  <main class="page-wrapper">
    <div
      ref="cardRef"
      class="glass-card"
      :class="colorMode.value"
      @mousemove="onMouseMove"
      @mouseleave="onMouseLeave"
    >
      <div class="card-topbar">
        <span class="status-dot"></span>
        <span class="status-text">disponible</span>
        <div class="topbar-right">
          <ColorModeButton />
        </div>
      </div>

      <AppHeader />
      <slot />
      <AppFooter />
    </div>
  </main>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Fira+Code:wght@300;400;500;600&family=Space+Grotesk:wght@300;400;600;700&display=swap");

.page-wrapper {
  position: relative;
  z-index: 1;
  min-height: 100vh;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem 1rem;
}

.glass-card {
  width: 100%;
  max-width: 520px;
  background: rgba(8, 8, 20, 0.65);
  backdrop-filter: blur(28px);
  -webkit-backdrop-filter: blur(28px);
  border: 1px solid rgba(0, 255, 238, 0.18);
  border-radius: 24px;
  box-shadow:
    0 0 0 1px rgba(0, 255, 238, 0.04) inset,
    0 4px 80px rgba(0, 0, 0, 0.6),
    0 0 60px rgba(0, 200, 255, 0.08);
  overflow: hidden;
  color: #e8e8ff;
  font-family: "Fira Code", monospace;
}

.card-topbar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 18px;
  border-bottom: 1px solid rgba(0, 255, 238, 0.08);
  background: rgba(0, 0, 0, 0.25);
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #00ff88;
  box-shadow: 0 0 8px #00ff88;
  animation: pulse-dot 2s ease-in-out infinite;
  flex-shrink: 0;
}

.status-text {
  font-size: 0.7rem;
  letter-spacing: 0.14em;
  color: #00ff88;
  text-transform: uppercase;
  font-family: "Fira Code", monospace;
}

.topbar-right {
  margin-left: auto;
}

@keyframes pulse-dot {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.35;
  }
}

/* MODO OSCURO — espacio profundo */
.glass-card.dark {
  background: rgba(3, 6, 25, 0.7);
  border-color: rgba(0, 255, 238, 0.15);
  box-shadow:
    0 0 0 1px rgba(0, 255, 238, 0.03) inset,
    0 4px 80px rgba(0, 0, 0, 0.7),
    0 0 80px rgba(0, 100, 255, 0.08);
  color: #e8e8ff;
}

/* MODO CLARO — amanecer espacial */
.glass-card.light {
  background: rgba(10, 22, 50, 0.6);
  border-color: rgba(255, 180, 50, 0.25);
  box-shadow:
    0 0 0 1px rgba(255, 180, 50, 0.05) inset,
    0 4px 80px rgba(0, 0, 0, 0.5),
    0 0 80px rgba(255, 120, 0, 0.1);
  color: #e8f4ff;
}

/* Topbar modo claro */
.glass-card.light .status-dot {
  background: #ffd700;
  box-shadow: 0 0 8px #ffd700;
}
.glass-card.light .status-text {
  color: #ffd700;
}
</style>
