<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import * as THREE from "three";
import { STLLoader } from "three/examples/jsm/loaders/STLLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

// ── Tipos ────────────────────────────────────────────
type PieceType = "K" | "Q" | "R" | "B" | "N" | "P";
type PieceColor = "w" | "b";
type Cell = { type: PieceType; color: PieceColor } | null;

// ── Lógica de ajedrez ────────────────────────────────
const createInitialBoard = (): Cell[][] => [
  ["R", "N", "B", "Q", "K", "B", "N", "R"].map((t) => ({
    type: t as PieceType,
    color: "b",
  })),
  Array.from({ length: 8 }, () => ({
    type: "P" as PieceType,
    color: "b" as PieceColor,
  })),
  Array(8).fill(null),
  Array(8).fill(null),
  Array(8).fill(null),
  Array(8).fill(null),
  Array.from({ length: 8 }, () => ({
    type: "P" as PieceType,
    color: "w" as PieceColor,
  })),
  ["R", "N", "B", "Q", "K", "B", "N", "R"].map((t) => ({
    type: t as PieceType,
    color: "w",
  })),
];

const getValidMoves = (
  board: Cell[][],
  row: number,
  col: number,
): [number, number][] => {
  const cell = board[row]?.[col];
  if (!cell) return [];
  const moves: [number, number][] = [];
  const { type, color } = cell;
  const inB = (r: number, c: number) => r >= 0 && r < 8 && c >= 0 && c < 8;
  const empty = (r: number, c: number) => inB(r, c) && !board[r]?.[c];
  const enemy = (r: number, c: number) => {
    if (!inB(r, c)) return false;
    const target = board[r]?.[c];
    return !!target && target.color !== color;
  };
  const canGo = (r: number, c: number) => empty(r, c) || enemy(r, c);
  const slide = (dr: number, dc: number) => {
    let r = row + dr,
      c = col + dc;
    while (inB(r, c)) {
      if (empty(r, c)) {
        moves.push([r, c]);
        r += dr;
        c += dc;
      } else {
        if (enemy(r, c)) moves.push([r, c]);
        break;
      }
    }
  };
  switch (type) {
    case "P": {
      const d = color === "w" ? -1 : 1;
      const sr = color === "w" ? 6 : 1;
      if (empty(row + d, col)) {
        moves.push([row + d, col]);
        if (row === sr && empty(row + 2 * d, col))
          moves.push([row + 2 * d, col]);
      }
      if (enemy(row + d, col - 1)) moves.push([row + d, col - 1]);
      if (enemy(row + d, col + 1)) moves.push([row + d, col + 1]);
      break;
    }
    case "R":
      slide(1, 0);
      slide(-1, 0);
      slide(0, 1);
      slide(0, -1);
      break;
    case "B":
      slide(1, 1);
      slide(1, -1);
      slide(-1, 1);
      slide(-1, -1);
      break;
    case "Q":
      slide(1, 0);
      slide(-1, 0);
      slide(0, 1);
      slide(0, -1);
      slide(1, 1);
      slide(1, -1);
      slide(-1, 1);
      slide(-1, -1);
      break;
    case "N": {
      const knightMoves: [number, number][] = [
        [-2, -1],
        [-2, 1],
        [-1, -2],
        [-1, 2],
        [1, -2],
        [1, 2],
        [2, -1],
        [2, 1],
      ];
      knightMoves.forEach(([dr, dc]) => {
        if (canGo(row + dr, col + dc)) moves.push([row + dr, col + dc]);
      });
      break;
    }
    case "K": {
      const kingMoves: [number, number][] = [
        [-1, -1],
        [-1, 0],
        [-1, 1],
        [0, -1],
        [0, 1],
        [1, -1],
        [1, 0],
        [1, 1],
      ];
      kingMoves.forEach(([dr, dc]) => {
        if (canGo(row + dr, col + dc)) moves.push([row + dr, col + dc]);
      });
      break;
    }
  }
  return moves;
};

// ── Estado del componente ────────────────────────────
const canvasRef = ref<HTMLCanvasElement | null>(null);
const statusMsg = ref("Turno de las Blancas ♔");
const capturedW = ref<string[]>([]);
const capturedB = ref<string[]>([]);
const loading = ref(true);

let renderer: THREE.WebGLRenderer;
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let controls: OrbitControls;
let animId: number;
let raycaster: THREE.Raycaster;
let mouse: THREE.Vector2;

let board = createInitialBoard();
let currentTurn: PieceColor = "w";
let selectedCell: [number, number] | null = null;
let validMoves: [number, number][] = [];

const squareMeshes: THREE.Mesh[][] = Array.from({ length: 8 }, () => Array(8));
const pieceMeshes: (THREE.Mesh | null)[][] = Array.from({ length: 8 }, () =>
  Array(8).fill(null),
);
const highlightGroup = new THREE.Group();
const pieceGeometries: Partial<Record<PieceType, THREE.BufferGeometry>> = {};

const symbols: Record<PieceType, string> = {
  K: "♔",
  Q: "♕",
  R: "♖",
  B: "♗",
  N: "♘",
  P: "♙",
};

// Materiales
const matDark = new THREE.MeshStandardMaterial({
  color: 0x020418,
  metalness: 0.9,
  roughness: 0.15,
});
const matLight = new THREE.MeshStandardMaterial({
  color: 0x071330,
  metalness: 0.7,
  roughness: 0.25,
  emissive: new THREE.Color(0x001040),
  emissiveIntensity: 0.4,
});
const matSel = new THREE.MeshStandardMaterial({
  color: 0x00ffee,
  transparent: true,
  opacity: 0.55,
  emissive: new THREE.Color(0x00ffee),
  emissiveIntensity: 1,
});
const matMove = new THREE.MeshStandardMaterial({
  color: 0x00ff88,
  transparent: true,
  opacity: 0.45,
  emissive: new THREE.Color(0x00ff88),
  emissiveIntensity: 0.8,
});
const matCapture = new THREE.MeshStandardMaterial({
  color: 0xff3355,
  transparent: true,
  opacity: 0.55,
  emissive: new THREE.Color(0xff2244),
  emissiveIntensity: 0.8,
});
const matWhite = new THREE.MeshStandardMaterial({
  color: 0x082040,
  metalness: 0.95,
  roughness: 0.05,
  emissive: new THREE.Color(0x00aaff),
  emissiveIntensity: 0.25,
});
const matBlack = new THREE.MeshStandardMaterial({
  color: 0x1a0535,
  metalness: 0.95,
  roughness: 0.05,
  emissive: new THREE.Color(0x7c3aed),
  emissiveIntensity: 0.25,
});

const bToW = (r: number, c: number) => ({ x: c - 3.5, z: r - 3.5 });

onMounted(async () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  // Escena
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x02030f);
  scene.fog = new THREE.FogExp2(0x02030f, 0.025);

  camera = new THREE.PerspectiveCamera(
    50,
    canvas.clientWidth / canvas.clientHeight,
    0.1,
    200,
  );
  camera.position.set(0, 11, 9);

  renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
  renderer.setSize(canvas.clientWidth, canvas.clientHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;

  // Luces
  scene.add(new THREE.AmbientLight(0x0a1030, 4));
  const sun = new THREE.DirectionalLight(0x00eeff, 5);
  sun.position.set(6, 12, 6);
  sun.castShadow = true;
  sun.shadow.mapSize.setScalar(2048);
  scene.add(sun);
  const fill = new THREE.DirectionalLight(0x7c3aed, 3);
  fill.position.set(-6, 4, -6);
  scene.add(fill);
  const top = new THREE.PointLight(0x00ffee, 2, 30);
  top.position.set(0, 10, 0);
  scene.add(top);

  // Estrellas
  const sGeo = new THREE.BufferGeometry();
  const sPos = new Float32Array(5000 * 3);
  for (let i = 0; i < 5000 * 3; i++) sPos[i] = (Math.random() - 0.5) * 300;
  sGeo.setAttribute("position", new THREE.BufferAttribute(sPos, 3));
  scene.add(
    new THREE.Points(
      sGeo,
      new THREE.PointsMaterial({
        color: 0xffffff,
        size: 0.07,
        transparent: true,
        opacity: 0.85,
      }),
    ),
  );

  // Polvo estelar azul
  const dGeo = new THREE.BufferGeometry();
  const dPos = new Float32Array(600 * 3);
  for (let i = 0; i < 600 * 3; i++) dPos[i] = (Math.random() - 0.5) * 80;
  dGeo.setAttribute("position", new THREE.BufferAttribute(dPos, 3));
  const dustPoints = new THREE.Points(
    dGeo,
    new THREE.PointsMaterial({
      color: 0x00aaff,
      size: 0.1,
      transparent: true,
      opacity: 0.3,
    }),
  );
  scene.add(dustPoints);

  // Marco del tablero
  const frame = new THREE.Mesh(
    new THREE.BoxGeometry(9.6, 0.2, 9.6),
    new THREE.MeshStandardMaterial({
      color: 0x04081f,
      metalness: 0.95,
      roughness: 0.08,
      emissive: new THREE.Color(0x00ffee),
      emissiveIntensity: 0.03,
    }),
  );
  frame.position.y = -0.12;
  frame.receiveShadow = true;
  scene.add(frame);

  // Bordes brillantes del tablero
  const edgePts = [
    new THREE.Vector3(-4.02, 0.06, -4.02),
    new THREE.Vector3(4.02, 0.06, -4.02),
    new THREE.Vector3(4.02, 0.06, -4.02),
    new THREE.Vector3(4.02, 0.06, 4.02),
    new THREE.Vector3(4.02, 0.06, 4.02),
    new THREE.Vector3(-4.02, 0.06, 4.02),
    new THREE.Vector3(-4.02, 0.06, 4.02),
    new THREE.Vector3(-4.02, 0.06, -4.02),
  ];
  scene.add(
    new THREE.LineSegments(
      new THREE.BufferGeometry().setFromPoints(edgePts),
      new THREE.LineBasicMaterial({
        color: 0x00ffee,
        transparent: true,
        opacity: 0.5,
      }),
    ),
  );

  // Casillas
  const sqGeo = new THREE.BoxGeometry(1, 0.08, 1);
  for (let r = 0; r < 8; r++) {
    for (let c = 0; c < 8; c++) {
      const mesh = new THREE.Mesh(
        sqGeo,
        (r + c) % 2 === 0 ? matDark : matLight,
      );
      const { x, z } = bToW(r, c);
      mesh.position.set(x, 0, z);
      mesh.receiveShadow = true;
      mesh.userData = { type: "square", row: r, col: c };
      squareMeshes[r]![c] = mesh;
      scene.add(mesh);
    }
  }

  // Grupo de highlights
  scene.add(highlightGroup);

  // Controls
  controls = new OrbitControls(camera, canvas);
  controls.enableDamping = true;
  controls.dampingFactor = 0.07;
  controls.maxPolarAngle = Math.PI / 2.1;
  controls.minDistance = 5;
  controls.maxDistance = 22;
  controls.target.set(0, 0, 0);

  raycaster = new THREE.Raycaster();
  mouse = new THREE.Vector2();

  // Cargar STLs con fallback
  const loader = new STLLoader();
  const fallbackGeos: Record<PieceType, THREE.BufferGeometry> = {
    K: new THREE.ConeGeometry(0.22, 0.75, 8),
    Q: new THREE.ConeGeometry(0.2, 0.68, 8),
    R: new THREE.CylinderGeometry(0.18, 0.22, 0.5, 8),
    B: new THREE.ConeGeometry(0.16, 0.62, 6),
    N: new THREE.BoxGeometry(0.26, 0.52, 0.18),
    P: new THREE.CylinderGeometry(0.16, 0.18, 0.38, 8),
  };

  await Promise.all(
    (["K", "Q", "R", "B", "N", "P"] as PieceType[]).map(
      (key) =>
        new Promise<void>((resolve) => {
          loader.load(
            `/chess/${key === "K" ? "king" : key === "Q" ? "queen" : key === "R" ? "rook" : key === "B" ? "bishop" : key === "N" ? "knight" : "pawn"}.stl`,
            (geo) => {
              geo.computeVertexNormals();
              geo.center();
              const box = new THREE.Box3().setFromObject(new THREE.Mesh(geo));
              const sz = box.getSize(new THREE.Vector3());
              const sc = 0.72 / Math.max(sz.x, sz.y, sz.z);
              geo.scale(sc, sc, sc);
              geo.rotateX(-Math.PI / 2);
              pieceGeometries[key] = geo;
              resolve();
            },
            undefined,
            () => {
              pieceGeometries[key] = fallbackGeos[key];
              resolve();
            },
          );
        }),
    ),
  );

  loading.value = false;
  renderBoard();

  // Loop
  let t = 0;
  const animate = () => {
    animId = requestAnimationFrame(animate);
    t += 0.016;
    dustPoints.rotation.y = t * 0.04;
    // Pulso en highlights
    highlightGroup.children.forEach((m) => {
      const mat = (m as THREE.Mesh).material as THREE.MeshStandardMaterial;
      if (mat.transparent) mat.opacity = 0.35 + Math.sin(t * 3) * 0.15;
    });
    controls.update();
    renderer.render(scene, camera);
  };
  animate();

  canvas.addEventListener("click", onClick);
  window.addEventListener("resize", onResize);
});

onUnmounted(() => {
  cancelAnimationFrame(animId);
  const canvas = canvasRef.value;
  if (canvas) {
    canvas.removeEventListener("click", onClick);
  }
  window.removeEventListener("resize", onResize);
  renderer?.dispose();
});

// ── Renderizar tablero ───────────────────────────────
const renderBoard = () => {
  console.log("renderBoard llamado");
  
  for (let r = 0; r < 8; r++)
    for (let c = 0; c < 8; c++) {
      const existingPiece = pieceMeshes[r]?.[c];
      if (existingPiece) {
        scene.remove(existingPiece);
        pieceMeshes[r]![c] = null;
      }
    }

  let piecesRendered = 0;
  for (let r = 0; r < 8; r++)
    for (let c = 0; c < 8; c++) {
      const cell = board[r]?.[c];
      if (!cell) continue;

      piecesRendered++;
      const geo = pieceGeometries[cell.type];
      if (!geo) continue;

      const mesh = new THREE.Mesh(
        geo,
        (cell.color === "w" ? matWhite : matBlack).clone(),
      );
      const { x, z } = bToW(r, c);
      mesh.position.set(x, 0.38, z);
      mesh.castShadow = true;
      mesh.userData = { type: "piece", row: r, col: c };
      pieceMeshes[r]![c] = mesh;
      scene.add(mesh);
    }
  
  console.log("Piezas renderizadas:", piecesRendered);
};

// ── Highlights ───────────────────────────────────────
const clearHighlights = () => {
  highlightGroup.clear();
  selectedCell = null;
  validMoves = [];
};
const showSelection = (row: number, col: number, moves: [number, number][]) => {
  highlightGroup.clear();
  validMoves = [];
  const hlGeo = new THREE.BoxGeometry(0.93, 0.012, 0.93);

  const addHL = (
    r: number,
    c: number,
    mat: THREE.MeshStandardMaterial,
    type?: string,
    mr?: number,
    mc?: number,
  ) => {
    const m = new THREE.Mesh(hlGeo, mat.clone());
    const pos = bToW(r, c);
    m.position.set(pos.x, 0.06, pos.z); // <-- subí a 0.06 para estar más arriba
    if (type) m.userData = { type, row: mr ?? r, col: mc ?? c };
    highlightGroup.add(m);
  };

  addHL(row, col, matSel);
  moves.forEach(([r, c]) => {
    const targetCell = board[r]?.[c];
    addHL(r, c, targetCell ? matCapture : matMove, "validMove", r, c);
  });
};

// ── Movimiento ───────────────────────────────────────
const doMove = (fr: number, fc: number, tr: number, tc: number) => {
  console.log("doMove llamado:", {from: [fr, fc], to: [tr, tc]});
  
  const captured = board[tr]?.[tc];
  if (captured) {
    console.log("Pieza capturada:", captured);
    const sym = symbols[captured.type];
    if (captured.color === "w") {
      capturedW.value.push(sym);
    } else {
      capturedB.value.push(sym);
    }
  }
  
  const sourcePiece = board[fr]?.[fc];
  console.log("Pieza origen:", sourcePiece);
  
  board[tr]![tc] = sourcePiece ?? null;
  board[fr]![fc] = null;
  
  console.log("Board actualizado:", board);
  
  currentTurn = currentTurn === "w" ? "b" : "w";
  statusMsg.value =
    currentTurn === "w" ? "Turno de las Blancas ♔" : "Turno de las Negras ♚";
  clearHighlights();
  renderBoard();
  
  console.log("Turno ahora:", currentTurn);
};

// ── Click ────────────────────────────────────────────
const onClick = (e: MouseEvent) => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const rect = canvas.getBoundingClientRect();
  mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);

  // ORDEN IMPORTANTE: highlights primero, luego piezas, luego casillas
  const targets = [
    ...(highlightGroup.children as THREE.Mesh[]),
    ...(pieceMeshes.flat().filter(Boolean) as THREE.Mesh[]),
    ...(squareMeshes.flat().filter(Boolean) as THREE.Mesh[]),
  ];

  const hits = raycaster.intersectObjects(targets);
  console.log("Hits encontrados:", hits.length, hits.map(h => ({type: (h.object as any).userData?.type, distance: h.distance})));
  if (!hits.length) return;

  const hit = hits[0];
  if (!hit) return;

  const obj = hit.object;
  if (!obj || !obj.userData) return;

  const userData = obj.userData as { type?: string; row?: number; col?: number };
  const { type, row, col } = userData;

  console.log("Click detectado:", {type, row, col, selectedCell});

  if (type === undefined || row === undefined || col === undefined) return;

  // Si es un highlight de movimiento válido
  if (type === "validMove" && selectedCell) {
    console.log("Moviendo pieza:", selectedCell, "a", [row, col]);
    doMove(selectedCell[0], selectedCell[1], row, col);
    return;
  }

  // Si es una pieza
  if (type === "piece") {
    const cell = board[row]?.[col];
    if (!cell) return;

    // Si es del color actual, seleccionar
    if (cell.color === currentTurn) {
      selectedCell = [row, col];
      validMoves = getValidMoves(board, row, col);
      showSelection(row, col, validMoves);
      return;
    }

    // Si es una pieza enemiga y tenemos selección, intentar capturar
    if (selectedCell && validMoves.some(([r, c]) => r === row && c === col)) {
      doMove(selectedCell[0], selectedCell[1], row, col);
      return;
    }

    clearHighlights();
    return;
  }

  // Si es una casilla
  if (type === "square") {
    if (selectedCell && validMoves.some(([r, c]) => r === row && c === col)) {
      doMove(selectedCell[0], selectedCell[1], row, col);
    } else {
      clearHighlights();
    }
  }
};

const onResize = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  camera.aspect = canvas.clientWidth / canvas.clientHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(canvas.clientWidth, canvas.clientHeight);
};

const restart = () => {
  board = createInitialBoard();
  currentTurn = "w";
  capturedW.value = [];
  capturedB.value = [];
  statusMsg.value = "Turno de las Blancas ♔";
  clearHighlights();
  renderBoard();
};
</script>

<template>
  <div class="chess-wrap">
    <!-- Cargando -->
    <div v-if="loading" class="chess-loading">
      <div class="chess-spinner"></div>
      <p>Cargando piezas espaciales...</p>
    </div>

    <!-- HUD -->
    <div v-else class="chess-hud">
      <div class="hud-turn">{{ statusMsg }}</div>
      <div class="hud-captured">
        <div class="cap-row">
          <span class="cap-label">♔ Capturadas:</span>
          <span class="cap-pieces cyan">{{ capturedB.join(" ") || "—" }}</span>
        </div>
        <div class="cap-row">
          <span class="cap-label">♚ Capturadas:</span>
          <span class="cap-pieces purple">{{
            capturedW.join(" ") || "—"
          }}</span>
        </div>
      </div>
      <button class="btn-restart" @click="restart">↺ Nueva partida</button>
    </div>

    <canvas ref="canvasRef" class="chess-canvas" />

    <div class="chess-legend">
      <span><span class="dot cyan-dot"></span>Blancas</span>
      <span><span class="dot purple-dot"></span>Negras</span>
      <span>🖱 Arrastrar para rotar · Click para mover</span>
    </div>
  </div>
</template>

<style scoped>
.chess-wrap {
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  flex-direction: column;
  background: #02030f;
}
.chess-canvas {
  flex: 1;
  width: 100%;
  display: block;
  cursor: crosshair;
}

.chess-loading {
  position: absolute;
  inset: 0;
  z-index: 10;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  background: #02030f;
}
.chess-loading p {
  color: #00ffee;
  font-family: "Orbitron", monospace;
  font-size: 0.85rem;
  letter-spacing: 0.1em;
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

.chess-hud {
  position: absolute;
  top: 1rem;
  left: 1rem;
  z-index: 10;
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
}
.hud-turn {
  font-family: "Orbitron", monospace;
  font-size: 0.82rem;
  color: #00ffee;
  letter-spacing: 0.1em;
  background: rgba(0, 0, 0, 0.65);
  border: 1px solid rgba(0, 255, 238, 0.2);
  border-radius: 10px;
  padding: 0.5rem 1rem;
  backdrop-filter: blur(10px);
}
.hud-captured {
  background: rgba(0, 0, 0, 0.55);
  border: 1px solid rgba(0, 255, 238, 0.12);
  border-radius: 10px;
  padding: 0.6rem 1rem;
  backdrop-filter: blur(10px);
  font-family: "Fira Code", monospace;
  font-size: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}
.cap-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.cap-label {
  color: rgba(160, 160, 220, 0.55);
  font-size: 0.68rem;
}
.cap-pieces {
  font-size: 0.88rem;
}
.cap-pieces.cyan {
  color: #00ffee;
}
.cap-pieces.purple {
  color: #7c3aed;
}

.btn-restart {
  background: rgba(124, 58, 237, 0.1);
  border: 1px solid rgba(124, 58, 237, 0.3);
  color: #a78bfa;
  padding: 0.45rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  font-family: "Fira Code", monospace;
  font-size: 0.78rem;
  letter-spacing: 0.08em;
  transition: all 0.2s;
}
.btn-restart:hover {
  background: rgba(124, 58, 237, 0.22);
  box-shadow: 0 0 12px rgba(124, 58, 237, 0.3);
}

.chess-legend {
  position: absolute;
  bottom: 0.8rem;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 1.5rem;
  align-items: center;
  font-family: "Fira Code", monospace;
  font-size: 0.7rem;
  color: rgba(160, 160, 220, 0.5);
  pointer-events: none;
  white-space: nowrap;
}
.dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  margin-right: 5px;
}
.cyan-dot {
  background: #00ffee;
  box-shadow: 0 0 6px #00ffee;
}
.purple-dot {
  background: #7c3aed;
  box-shadow: 0 0 6px #7c3aed;
}
</style>
