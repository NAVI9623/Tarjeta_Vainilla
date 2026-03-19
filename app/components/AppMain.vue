<template>
  <div class="cv-body">
    <div class="divider"></div>

    <section class="cv-section">
      <h3 class="section-title">Sobre mí</h3>
      <p class="section-text">
        Desarrollador con experiencia en Análisis de Datos, Ciberseguridad y
        entrenamiento de modelos de Machine Learning. Apasionado por transformar
        datos en soluciones inteligentes, seguras y escalables.
      </p>
    </section>

    <div class="divider"></div>

    <section class="cv-section">
      <h3 class="section-title">Habilidades</h3>
      <div class="tags">
        <span>HTML5</span>
        <span>CSS3</span>
        <span>JavaScript</span>
        <span>Python</span>
        <span>Vue.js</span>
        <span>Git</span>
        <span>Machine Learning</span>
        <span>Análisis de Datos</span>
        <span>Ética Hacker</span>
        <span>Ciberseguridad</span>
      </div>
    </section>
  </div>
</template>

<style scoped>
.cv-body {
  padding: 0.5rem 2rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.4rem;
}

.divider {
  height: 1px;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(0, 255, 238, 0.2),
    transparent
  );
}

.cv-section {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.section-title {
  font-size: 0.68rem;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #00ffee;
  font-family: "Fira Code", monospace;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 0;
}

.section-title::after {
  content: "";
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, rgba(0, 255, 238, 0.2), transparent);
}

.section-text {
  font-size: 0.85rem;
  line-height: 1.8;
  color: rgba(180, 180, 220, 0.8);
  font-family: "Space Grotesk", sans-serif;
  margin: 0;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tags span {
  background: rgba(0, 255, 238, 0.06);
  border: 1px solid rgba(0, 255, 238, 0.22);
  color: #00ffee;
  padding: 5px 12px;
  border-radius: 6px;
  font-size: 0.75rem;
  font-family: "Fira Code", monospace;
  letter-spacing: 0.04em;
  transition: all 0.2s ease;
  cursor: default;
}

.tags span:hover {
  background: rgba(0, 255, 238, 0.14);
  box-shadow: 0 0 12px rgba(0, 255, 238, 0.2);
  transform: translateY(-2px);
}
</style>
