<script setup lang="ts">
import { defineAsyncComponent } from "vue";

// ✅ CODE SPLITTING — Three.js solo se descarga cuando este componente se monta
const ModelViewer = defineAsyncComponent({
  loader: () => import("~/components/ModelViewer.vue"),
  loadingComponent: {
    template: `
      <div class="model-loading">
        <div class="model-spinner"></div>
      </div>
    `,
  },
  delay: 200,
  timeout: 10000,
});
</script>

<template>
  <div class="profile-header">
    <!-- Visor 3D — reemplaza el avatar -->
    <div class="avatar-frame">
      <ClientOnly>
        <ModelViewer stl-path="/models/model.stl" class="model-wrap" />
        <template #fallback>
          <div class="model-loading">
            <div class="model-spinner"></div>
          </div>
        </template>
      </ClientOnly>
    </div>

    <h1 class="profile-name">Ivan Ignacio Ramirez Montañez</h1>
    <p class="profile-role">Datos · Ciberseguridad · Machine Learning</p>
  </div>
</template>

<style scoped>
.profile-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1.5rem 2rem 1rem;
  text-align: center;
  gap: 0.8rem;
}

.avatar-frame {
  width: 220px;
  height: 220px;
  position: relative;
}

.model-wrap {
  width: 100%;
  height: 100%;
}

.model-loading {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.model-spinner {
  width: 48px;
  height: 48px;
  border: 3px solid rgba(0, 255, 238, 0.15);
  border-top-color: #00ffee;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.profile-name {
  font-family: "Space Grotesk", sans-serif;
  font-size: 1.25rem;
  font-weight: 700;
  color: #fff;
  margin: 0;
}

.profile-role {
  font-size: 0.72rem;
  letter-spacing: 0.14em;
  color: #00ffee;
  text-transform: uppercase;
  font-family: "Fira Code", monospace;
  margin: 0;
  opacity: 0.85;
}
</style>
