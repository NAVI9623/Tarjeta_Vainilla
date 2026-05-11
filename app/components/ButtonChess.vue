<script setup lang="ts">
defineEmits<{ open: [] }>();
</script>

<template>
  <button class="Btn" @click="$emit('open')" title="Ajedrez Espacial">
    <span class="svgContainer">
      <svg fill="white" viewBox="0 0 24 24" height="1.4em">
        <path
          d="M19 22H5v-2h14v2M8.5 8H11V3H9V1h6v2h-2v5h2.5L14 10h-1v2l2 2v2H9v-2l2-2v-2H10L8.5 8z"
        />
      </svg>
    </span>
    <span class="BG"></span>
  </button>
</template>

<style scoped>
.Btn {
  width: 45px;
  height: 45px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  background-color: transparent;
  position: relative;
  border-radius: 7px;
  cursor: pointer;
  transition: all 0.3s;
}
.svgContainer {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  border-radius: 10px;
  transition: all 0.3s;
  border: 1px solid rgba(124, 58, 237, 0.4);
}
.BG {
  position: absolute;
  width: 100%;
  height: 100%;
  background: #0d0020;
  z-index: -1;
  border-radius: 10px;
  transition: all 0.3s;
}
.Btn:hover .BG {
  transform: rotate(35deg);
  transform-origin: bottom;
}
.Btn:hover .svgContainer {
  background-color: rgba(124, 58, 237, 0.15);
  backdrop-filter: blur(4px);
  box-shadow: 0 0 16px rgba(124, 58, 237, 0.3);
}
</style>
