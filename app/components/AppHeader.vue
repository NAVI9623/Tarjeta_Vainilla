<script setup lang="ts"></script>

<template>
  <div class="profile">
    <div class="avatar">
      <img src="/profile/profile.jpg" alt="profile" />
    </div>
    <h1 id="nombre">Ramirez Montañez Ivan Ignacio</h1>
  </div>
</template>

<style scoped>
.profile {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  margin-top: 2rem;
}

.avatar {
  width: 80px;
  height: 80px;
  overflow: hidden;
  border-radius: 50%;
}

.avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

#nombre {
  margin: 12px 0 0;
}
</style>
