<template>
  <div class="profile-header">
    <div class="avatar-frame">
      <SvgFrame class="svg-deco" />
      <div class="avatar">
        <img src="/profile/profile.jpg" alt="Ivan" />
      </div>
    </div>
    <h1 class="profile-name">Ivan Ignacio Ramirez Montañez</h1>
    <p class="profile-role">
      Full Stack Developer. Diseñador web. Backend Engineer
    </p>
  </div>
</template>

<style scoped>
.profile-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2.5rem 2rem 1.5rem;
  text-align: center;
  gap: 0.8rem;
}

.avatar-frame {
  position: relative;
  width: 140px;
  height: 140px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.svg-deco {
  position: absolute;
  inset: -30px;
  width: calc(100% + 60px);
  height: calc(100% + 60px);
  opacity: 0.85;
  pointer-events: none;
}

.avatar {
  width: 90px;
  height: 90px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid #00ffee;
  box-shadow:
    0 0 20px rgba(0, 255, 238, 0.4),
    0 0 40px rgba(0, 255, 238, 0.1);
  position: relative;
  z-index: 1;
  background: rgba(0, 255, 238, 0.05);
}

.avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.profile-name {
  font-family: "Space Grotesk", sans-serif;
  font-size: 1.25rem;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.01em;
  line-height: 1.3;
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
