<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import * as THREE from "three";
import { STLLoader } from "three/examples/jsm/loaders/STLLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const props = defineProps<{
  stlPath?: string;
  width?: number;
  height?: number;
}>();

const canvasRef = ref<HTMLCanvasElement | null>(null);
let renderer: THREE.WebGLRenderer;
let animId: number;
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let controls: OrbitControls;
let mesh: THREE.Mesh;

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const w = props.width ?? canvas.parentElement?.clientWidth ?? 300;
  const h = props.height ?? canvas.parentElement?.clientHeight ?? 300;

  // ── Escena ──────────────────────────────────────────
  scene = new THREE.Scene();

  // ── Cámara ──────────────────────────────────────────
  camera = new THREE.PerspectiveCamera(45, w / h, 0.1, 1000);
  camera.position.set(0, 0, 5);

  // ── Renderer ────────────────────────────────────────
  renderer = new THREE.WebGLRenderer({
    canvas,
    antialias: true,
    alpha: true, // fondo transparente
  });
  renderer.setSize(w, h);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.toneMapping = THREE.ACESFilmicToneMapping;

  //   // ── Luces ───────────────────────────────────────────
  //   const ambient = new THREE.AmbientLight(0x404040, 2);
  //   scene.add(ambient);

  //   const dirLight1 = new THREE.DirectionalLight(0x00ffee, 3);
  //   dirLight1.position.set(5, 5, 5);
  //   scene.add(dirLight1);

  //   const dirLight2 = new THREE.DirectionalLight(0xbc5ee7, 2);
  //   dirLight2.position.set(-5, -3, -5);
  //   scene.add(dirLight2);

  //   const pointLight = new THREE.PointLight(0x00ffee, 2, 20);
  //   pointLight.position.set(0, 3, 3);
  //   scene.add(pointLight);

  // ── Material más visible ─────────────────────────────
  const material = new THREE.MeshStandardMaterial({
    color: 0x2a4a7f, // azul acero base
    metalness: 0.85,
    roughness: 0.15,
    emissive: new THREE.Color(0x00aaff),
    emissiveIntensity: 0.4, // sube esto si quieres más brillo
  });

  const wireMaterial = new THREE.MeshBasicMaterial({
    color: 0x00ffee,
    wireframe: true,
    transparent: true,
    opacity: 0.12, // un poco más visible
  });

  // ── Luces más fuertes ────────────────────────────────
  const ambient = new THREE.AmbientLight(0x334466, 3);
  scene.add(ambient);

  const dirLight1 = new THREE.DirectionalLight(0x00eeff, 6); // cyan fuerte
  dirLight1.position.set(3, 5, 5);
  scene.add(dirLight1);

  const dirLight2 = new THREE.DirectionalLight(0x7c3aed, 4); // púrpura lateral
  dirLight2.position.set(-5, -2, -3);
  scene.add(dirLight2);

  const dirLight3 = new THREE.DirectionalLight(0xffffff, 3); // blanco frontal
  dirLight3.position.set(0, 2, 8);
  scene.add(dirLight3);

  const pointLight = new THREE.PointLight(0x00ffee, 4, 15); // brillo puntual
  pointLight.position.set(0, 3, 3);
  scene.add(pointLight);

  const rimLight = new THREE.PointLight(0xff6600, 2, 10); // rim naranja
  rimLight.position.set(-3, -3, -2);
  scene.add(rimLight);

  // ── Cargar STL o geometría por defecto ───────────────
  if (props.stlPath) {
    const loader = new STLLoader();
    loader.load(props.stlPath, (geometry) => {
      geometry.computeVertexNormals();

      // Centrar y escalar automáticamente
      geometry.center();
      const box = new THREE.Box3().setFromObject(new THREE.Mesh(geometry));
      const size = box.getSize(new THREE.Vector3());
      const maxDim = Math.max(size.x, size.y, size.z);
      const scale = 3 / maxDim;

      mesh = new THREE.Mesh(geometry, material);
      mesh.scale.setScalar(scale);
      scene.add(mesh);

      const wireMesh = new THREE.Mesh(geometry, wireMaterial);
      wireMesh.scale.setScalar(scale);
      scene.add(wireMesh);
    });
  } else {
    // Geometría de respaldo — gema icosaédrica
    const geometry = new THREE.IcosahedronGeometry(1.5, 1);
    mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);
    scene.add(new THREE.Mesh(geometry, wireMaterial));
  }

  // ── Partículas flotantes ─────────────────────────────
  const particleGeo = new THREE.BufferGeometry();
  const count = 200;
  const positions = new Float32Array(count * 3);
  for (let i = 0; i < count * 3; i++) {
    positions[i] = (Math.random() - 0.5) * 12;
  }
  particleGeo.setAttribute("position", new THREE.BufferAttribute(positions, 3));
  const particleMat = new THREE.PointsMaterial({
    color: 0x00ffee,
    size: 0.04,
    transparent: true,
    opacity: 0.6,
  });
  scene.add(new THREE.Points(particleGeo, particleMat));

  // ── OrbitControls (rotar con mouse/touch) ───────────
  controls = new OrbitControls(camera, canvas);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.enableZoom = false;
  controls.autoRotate = true;
  controls.autoRotateSpeed = 1.5;

  // ── Pausa autorotate al interactuar ─────────────────
  controls.addEventListener("start", () => {
    controls.autoRotate = false;
  });
  controls.addEventListener("end", () => {
    setTimeout(() => {
      controls.autoRotate = true;
    }, 2000);
  });

  // ── Loop de animación ────────────────────────────────
  const animate = () => {
    animId = requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  };
  animate();
});

onUnmounted(() => {
  cancelAnimationFrame(animId);
  renderer?.dispose();
});
</script>

<template>
  <canvas ref="canvasRef" class="model-canvas" />
</template>

<style scoped>
.model-canvas {
  width: 100%;
  height: 100%;
  display: block;
  cursor: grab;
}
.model-canvas:active {
  cursor: grabbing;
}
</style>
