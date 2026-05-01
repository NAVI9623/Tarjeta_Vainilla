<template>
  <canvas ref="canvasRef" class="space-bg" />
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from "vue";
import * as THREE from "three";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const colorMode = useColorMode();
let renderer: THREE.WebGLRenderer;
let animId: number;
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let dustPoints: THREE.Points;
let nebPoints: THREE.Points;

const DARK_BG = new THREE.Color(0x03040f);
const LIGHT_BG = new THREE.Color(0x0a1628);

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  scene = new THREE.Scene();
  scene.background = new THREE.Color(DARK_BG);
  scene.fog = new THREE.FogExp2(0x03040f, 0.025);

  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000,
  );
  camera.position.z = 5;

  renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(window.devicePixelRatio);

  // ── Estrellas ────────────────────────────────────────
  const starCount = 4000;
  const starPos = new Float32Array(starCount * 3);
  for (let i = 0; i < starCount * 3; i++) {
    starPos[i] = (Math.random() - 0.5) * 300;
  }
  const starGeo = new THREE.BufferGeometry();
  starGeo.setAttribute("position", new THREE.BufferAttribute(starPos, 3));
  const starMat = new THREE.PointsMaterial({
    color: 0xffffff,
    size: 0.08,
    transparent: true,
    opacity: 0.9,
  });
  scene.add(new THREE.Points(starGeo, starMat));

  // ── Polvo cian ───────────────────────────────────────
  const dustCount = 800;
  const dustPos = new Float32Array(dustCount * 3);
  for (let i = 0; i < dustCount * 3; i++) {
    dustPos[i] = (Math.random() - 0.5) * 60;
  }
  const dustGeo = new THREE.BufferGeometry();
  dustGeo.setAttribute("position", new THREE.BufferAttribute(dustPos, 3));
  const dustMat = new THREE.PointsMaterial({
    color: 0x00ffee,
    size: 0.12,
    transparent: true,
    opacity: 0.35,
  });
  dustPoints = new THREE.Points(dustGeo, dustMat);
  scene.add(dustPoints);

  // ── Nebulosa púrpura ─────────────────────────────────
  const nebCount = 400;
  const nebPos = new Float32Array(nebCount * 3);
  for (let i = 0; i < nebCount * 3; i++) {
    nebPos[i] = (Math.random() - 0.5) * 40;
  }
  const nebGeo = new THREE.BufferGeometry();
  nebGeo.setAttribute("position", new THREE.BufferAttribute(nebPos, 3));
  const nebMat = new THREE.PointsMaterial({
    color: 0x7c3aed,
    size: 0.18,
    transparent: true,
    opacity: 0.3,
  });
  nebPoints = new THREE.Points(nebGeo, nebMat);
  scene.add(nebPoints);

  // ── Mouse parallax ───────────────────────────────────
  let mouseX = 0,
    mouseY = 0;
  const onMouseMove = (e: MouseEvent) => {
    mouseX = (e.clientX / window.innerWidth - 0.5) * 0.6;
    mouseY = (e.clientY / window.innerHeight - 0.5) * 0.6;
  };
  window.addEventListener("mousemove", onMouseMove);

  // ── Resize ───────────────────────────────────────────
  const onResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  };
  window.addEventListener("resize", onResize);

  // ── Cambio de tema reactivo ──────────────────────────
  watch(
    () => colorMode.value,
    (mode) => {
      const isLight = mode === "light";

      // Fondo
      scene.background = isLight
        ? new THREE.Color(0x0a1628) // azul noche profundo en claro
        : new THREE.Color(0x03040f);

      scene.fog = isLight
        ? new THREE.FogExp2(0x0a1628, 0.02)
        : new THREE.FogExp2(0x03040f, 0.025);

      // Partículas cambian de color
      (dustPoints.material as THREE.PointsMaterial).color.set(
        isLight ? 0xffd700 : 0x00ffee, // dorado de día, cian de noche
      );
      (dustPoints.material as THREE.PointsMaterial).opacity = isLight
        ? 0.5
        : 0.35;
      (nebPoints.material as THREE.PointsMaterial).color.set(
        isLight ? 0xff6b35 : 0x7c3aed, // naranja amanecer de día, púrpura de noche
      );
      (nebPoints.material as THREE.PointsMaterial).opacity = isLight
        ? 0.4
        : 0.3;

      // Estrellas más visibles de día (paradójicamente más brillantes)
      starMat.opacity = isLight ? 0.6 : 0.9;
    },
    { immediate: true },
  );

  // ── Loop ─────────────────────────────────────────────
  let t = 0;
  const animate = () => {
    animId = requestAnimationFrame(animate);
    t += 0.0003;

    dustPoints.rotation.y = t * 0.4;
    dustPoints.rotation.x = t * 0.15;
    nebPoints.rotation.y = -t * 0.25;
    nebPoints.rotation.x = t * 0.1;

    camera.position.x += (mouseX - camera.position.x) * 0.025;
    camera.position.y += (-mouseY - camera.position.y) * 0.025;
    camera.lookAt(scene.position);

    renderer.render(scene, camera);
  };
  animate();

  onUnmounted(() => {
    cancelAnimationFrame(animId);
    window.removeEventListener("mousemove", onMouseMove);
    window.removeEventListener("resize", onResize);
    renderer.dispose();
  });
});
</script>

<style scoped>
.space-bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  transition: background 0.8s ease;
}
</style>
