<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import * as THREE from "three";
import { STLLoader } from "three/examples/jsm/loaders/STLLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const props = defineProps<{ stlPath?: string }>();

const canvasRef = ref<HTMLCanvasElement | null>(null);
const disintegrated = ref(false);
const animating = ref(false);
const hint = ref("⚡ Click para desintegrar");

let renderer: THREE.WebGLRenderer;
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let controls: OrbitControls;
let animId: number;

// Meshes sólidos
let solidMesh: THREE.Mesh | null = null;
let wireMesh: THREE.Mesh | null = null;

// Sistema de partículas
let particles: THREE.Points | null = null;
let origPos: Float32Array; // posiciones originales de cada partícula
let velocities: Float32Array; // dirección/velocidad aleatoria de cada partícula

// Estado de animación
let progress = 0; // 0 = ensamblado, 1 = desintegrado
let direction = 0; // 1 = dispersar, -1 = reensammblar
let driftTimer: ReturnType<typeof setTimeout> | null = null;

const SPEED = 0.016; // velocidad de la transición
const MAX_PART = 8000; // máximo de partículas para buen rendimiento

const easeInOut = (t: number) =>
  t < 0.5 ? 2 * t * t : 1 - Math.pow(-2 * t + 2, 2) / 2;

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const w = canvas.parentElement?.clientWidth ?? 300;
  const h = canvas.parentElement?.clientHeight ?? 300;

  // ── Escena ───────────────────────────────────────
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(45, w / h, 0.1, 1000);
  camera.position.set(0, 0, 5);

  renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
  renderer.setSize(w, h);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.toneMapping = THREE.ACESFilmicToneMapping;

  // ── Luces ────────────────────────────────────────
  scene.add(new THREE.AmbientLight(0x334466, 3));
  const l1 = new THREE.DirectionalLight(0x00eeff, 6);
  l1.position.set(3, 5, 5);
  scene.add(l1);
  const l2 = new THREE.DirectionalLight(0x7c3aed, 4);
  l2.position.set(-5, -2, -5);
  scene.add(l2);
  const l3 = new THREE.DirectionalLight(0xffffff, 3);
  l3.position.set(0, 2, 8);
  scene.add(l3);
  const pl = new THREE.PointLight(0x00ffee, 4, 15);
  pl.position.set(0, 3, 3);
  scene.add(pl);
  const rl = new THREE.PointLight(0xff6600, 2, 10);
  rl.position.set(-3, -3, -2);
  scene.add(rl);

  // ── Materiales ───────────────────────────────────
  const solidMat = new THREE.MeshStandardMaterial({
    color: 0x2a4a7f,
    metalness: 0.85,
    roughness: 0.15,
    emissive: new THREE.Color(0x00aaff),
    emissiveIntensity: 0.4,
  });
  const wireMat = new THREE.MeshBasicMaterial({
    color: 0x00ffee,
    wireframe: true,
    transparent: true,
    opacity: 0.12,
  });

  // ── OrbitControls ────────────────────────────────
  controls = new OrbitControls(camera, canvas);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.enableZoom = false;
  controls.autoRotate = true;
  controls.autoRotateSpeed = 1.5;
  controls.addEventListener("start", () => {
    controls.autoRotate = false;
  });
  controls.addEventListener("end", () =>
    setTimeout(() => {
      controls.autoRotate = true;
    }, 2000),
  );

  // ── Cargar STL ───────────────────────────────────
  const buildScene = (geo: THREE.BufferGeometry, scale: number) => {
    geo.computeVertexNormals();
    geo.center();

    solidMesh = new THREE.Mesh(geo, solidMat);
    solidMesh.scale.setScalar(scale);
    scene.add(solidMesh);

    wireMesh = new THREE.Mesh(geo, wireMat);
    wireMesh.scale.setScalar(scale);
    scene.add(wireMesh);

    buildParticles(geo, scale);
  };

  if (props.stlPath) {
    new STLLoader().load(
      props.stlPath,
      (geo) => {
        const box = new THREE.Box3().setFromObject(new THREE.Mesh(geo));
        const size = box.getSize(new THREE.Vector3());
        const scale = 3 / Math.max(size.x, size.y, size.z);
        buildScene(geo, scale);
      },
      undefined,
      () => buildScene(new THREE.IcosahedronGeometry(1.5, 2), 1),
    );
  } else {
    buildScene(new THREE.IcosahedronGeometry(1.5, 2), 1);
  }

  // ── Loop ─────────────────────────────────────────
  const animate = () => {
    animId = requestAnimationFrame(animate);
    controls.update();
    if (direction !== 0) tickParticles();
    else if (disintegrated.value) driftParticles();
    renderer.render(scene, camera);
  };
  animate();
});

onUnmounted(() => {
  cancelAnimationFrame(animId);
  if (driftTimer) clearTimeout(driftTimer);
  renderer?.dispose();
});

// ── Crear sistema de partículas ───────────────────────
const buildParticles = (geo: THREE.BufferGeometry, scale: number) => {
  // ✅ Fix: verificar que el atributo existe
  const srcAttr = geo.attributes["position"];
  if (!srcAttr) return;
  const src = srcAttr as THREE.BufferAttribute;

  const step = Math.max(1, Math.floor(src.count / MAX_PART));
  const count = Math.floor(src.count / step);

  origPos = new Float32Array(count * 3);
  velocities = new Float32Array(count * 3);
  const colors = new Float32Array(count * 3);
  const curPos = new Float32Array(count * 3);

  for (let i = 0; i < count; i++) {
    const vi = i * step;

    origPos[i * 3] = curPos[i * 3] = src.getX(vi) * scale;
    origPos[i * 3 + 1] = curPos[i * 3 + 1] = src.getY(vi) * scale;
    origPos[i * 3 + 2] = curPos[i * 3 + 2] = src.getZ(vi) * scale;

    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const spd = 1.5 + Math.random() * 3.5;
    velocities[i * 3] = Math.sin(phi) * Math.cos(theta) * spd;
    velocities[i * 3 + 1] = Math.sin(phi) * Math.sin(theta) * spd;
    velocities[i * 3 + 2] = Math.cos(phi) * spd;

    const t = Math.random();
    colors[i * 3] = t * 0.0 + (1 - t) * 0.49;
    colors[i * 3 + 1] = t * 1.0 + (1 - t) * 0.23;
    colors[i * 3 + 2] = t * 0.93 + (1 - t) * 0.93;
  }

  const pGeo = new THREE.BufferGeometry();
  pGeo.setAttribute("position", new THREE.BufferAttribute(curPos, 3));
  pGeo.setAttribute("color", new THREE.BufferAttribute(colors, 3));

  const pMat = new THREE.PointsMaterial({
    size: 0.045,
    vertexColors: true,
    transparent: true,
    opacity: 0,
    sizeAttenuation: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  });

  particles = new THREE.Points(pGeo, pMat);
  particles.visible = false;
  scene.add(particles);
};

// ── Actualizar posiciones en animación ────────────────
const tickParticles = () => {
  if (!particles || !origPos || !velocities) return; // ✅ Fix: guarda de existencia

  progress = Math.max(0, Math.min(1, progress + SPEED * direction));
  const eased = easeInOut(progress);

  // ✅ Fix: casteo explícito del atributo
  const posAttr = particles.geometry.getAttribute(
    "position",
  ) as THREE.BufferAttribute;
  const arr = posAttr.array as Float32Array;

  for (let i = 0; i < origPos.length / 3; i++) {
    arr[i * 3] = (origPos[i * 3] ?? 0) + (velocities[i * 3] ?? 0) * eased;
    arr[i * 3 + 1] =
      (origPos[i * 3 + 1] ?? 0) + (velocities[i * 3 + 1] ?? 0) * eased;
    arr[i * 3 + 2] =
      (origPos[i * 3 + 2] ?? 0) + (velocities[i * 3 + 2] ?? 0) * eased;
  }
  posAttr.needsUpdate = true;

  const mat = particles.material as THREE.PointsMaterial;
  mat.opacity =
    direction === 1 ? Math.min(1, progress * 2.5) : Math.max(0, progress * 2);

  if (progress >= 1 && direction === 1) {
    direction = 0;
    animating.value = false;
    hint.value = "↺ Click para reconstruir";
    driftTimer = setTimeout(triggerReassemble, 3500);
  }

  if (progress <= 0 && direction === -1) {
    direction = 0;
    animating.value = false;
    disintegrated.value = false;
    hint.value = "⚡ Click para desintegrar";
    particles!.visible = false;
    if (solidMesh) solidMesh.visible = true;
    if (wireMesh) wireMesh.visible = true;
  }
};

// ── Deriva sutil mientras está desintegrado ───────────
const driftParticles = () => {
  if (!particles || !origPos || !velocities) return; // ✅ Fix

  const t = Date.now() * 0.0006;
  const posAttr = particles.geometry.getAttribute(
    "position",
  ) as THREE.BufferAttribute;
  const arr = posAttr.array as Float32Array;

  for (let i = 0; i < origPos.length / 3; i++) {
    arr[i * 3] =
      (origPos[i * 3] ?? 0) +
      (velocities[i * 3] ?? 0) +
      Math.sin(t + i * 0.07) * 0.03;
    arr[i * 3 + 1] =
      (origPos[i * 3 + 1] ?? 0) +
      (velocities[i * 3 + 1] ?? 0) +
      Math.cos(t + i * 0.09) * 0.03;
    arr[i * 3 + 2] =
      (origPos[i * 3 + 2] ?? 0) +
      (velocities[i * 3 + 2] ?? 0) +
      Math.sin(t + i * 0.11) * 0.03;
  }
  posAttr.needsUpdate = true;
};

// ── Triggers ──────────────────────────────────────────
const triggerDisintegrate = () => {
  if (!particles || animating.value || disintegrated.value) return;

  animating.value = true;
  disintegrated.value = true;
  hint.value = "💫 Desintegrando...";

  if (solidMesh) solidMesh.visible = false;
  if (wireMesh) wireMesh.visible = false;
  particles.visible = true;
  direction = 1;
};

const triggerReassemble = () => {
  if (!particles || animating.value || !disintegrated.value) return;
  if (driftTimer) {
    clearTimeout(driftTimer);
    driftTimer = null;
  }

  animating.value = true;
  hint.value = "✨ Reconstruyendo...";
  direction = -1;
};

const toggle = () => {
  if (animating.value) return;
  disintegrated.value ? triggerReassemble() : triggerDisintegrate();
};
</script>

<template>
  <div class="model-outer" @click="toggle">
    <canvas ref="canvasRef" class="model-canvas" />
    <div class="model-hint" :class="{ glowing: disintegrated }">
      {{ hint }}
    </div>
  </div>
</template>

<style scoped>
.model-outer {
  position: relative;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.model-canvas {
  width: 100%;
  height: 100%;
  display: block;
}

.model-hint {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  font-family: "Fira Code", monospace;
  font-size: 0.65rem;
  letter-spacing: 0.1em;
  color: rgba(0, 255, 238, 0.5);
  text-transform: uppercase;
  white-space: nowrap;
  transition: all 0.4s ease;
  pointer-events: none;
}

.model-hint.glowing {
  color: #00ffee;
  text-shadow:
    0 0 8px rgba(0, 255, 238, 0.8),
    0 0 20px rgba(0, 255, 238, 0.4);
  animation: pulse-hint 1.5s ease-in-out infinite;
}

@keyframes pulse-hint {
  0%,
  100% {
    opacity: 0.7;
  }
  50% {
    opacity: 1;
  }
}
</style>
