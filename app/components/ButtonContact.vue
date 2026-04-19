<script setup lang="ts">
defineEmits<{ open: [] }>();
</script>

<template>
  <button class="Btn" @click="$emit('open')">
    <span class="svgContainer">
      <svg fill="white" viewBox="0 0 24 24" height="1.4em">
        <path
          d="M20 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"
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
  backdrop-filter: blur(0px);
  border-radius: 10px;
  transition: all 0.3s;
  border: 1px solid rgba(0, 255, 238, 0.3);
}
.BG {
  position: absolute;
  width: 100%;
  height: 100%;
  background: #0d1117;
  z-index: -1;
  border-radius: 10px;
  transition: all 0.3s;
}
.Btn:hover .BG {
  transform: rotate(35deg);
  transform-origin: bottom;
}
.Btn:hover .svgContainer {
  background-color: rgba(0, 255, 238, 0.12);
  backdrop-filter: blur(4px);
  box-shadow: 0 0 16px rgba(0, 255, 238, 0.2);
}
</style>
