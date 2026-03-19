<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";

const canvasRef = ref<HTMLCanvasElement | null>(null);
let animId = 0;

const kana =
  "アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲン0123456789";

onMounted(() => {
  const c = canvasRef.value;
  if (!c) return; // ✅ Fix: verifica que canvas existe

  const ctx = c.getContext("2d");
  if (!ctx) return; // ✅ Fix: verifica que ctx existe

  const fontSize = 15;
  let cols: number;
  let drops: number[];

  const resize = () => {
    c.width = window.innerWidth;
    c.height = window.innerHeight;
    cols = Math.floor(c.width / fontSize);
    drops = Array(cols).fill(1);
  };

  resize();
  window.addEventListener("resize", resize);

  const draw = () => {
    const isDark = document.documentElement.classList.contains("dark");
    ctx.fillStyle = isDark ? "rgba(3,3,8,0.06)" : "rgba(100,0,180,0.06)";
    ctx.fillRect(0, 0, c.width, c.height);

    drops.forEach((y, i) => {
      const bright = Math.random() > 0.95;
      ctx.fillStyle = bright
        ? "#ffffff"
        : isDark
          ? "rgba(0,255,180,0.75)"
          : "rgba(0,180,255,0.7)";
      ctx.font = `${fontSize}px monospace`;

      // ✅ Fix: garantiza que el char nunca sea undefined
      const char = kana[Math.floor(Math.random() * kana.length)] ?? "ア";
      ctx.fillText(char, i * fontSize, y * fontSize);

      if (y * fontSize > c.height && Math.random() > 0.975) drops[i] = 0;
      drops[i] = (drops[i] ?? 0) + 1; // ✅ Fix: fallback a 0 si es undefined
    });

    animId = requestAnimationFrame(draw);
  };

  draw();

  return () => {
    window.removeEventListener("resize", resize);
    cancelAnimationFrame(animId);
  };
});

onUnmounted(() => {
  cancelAnimationFrame(animId);
});
</script>
