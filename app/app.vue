<template>
  <body>
    <button id="toggle-theme">🌙 Modo Noche</button>

    Ahora quedará así:

    <button @click="toggleTheme">
      {{ darkMode ? "☀️ Modo Día" : "🌙 Modo Noche" }}
    </button>

    <div class="cv-container">
      <header>
        <h1>Iván Ignacio Ramírez Montañez</h1>
        <h2>Ingeniero en Tecnologías de la Información y Comunicaciones</h2>
        <p class="profile">
          Ingeniero en TICS con experiencia en desarrollo web, automatización de
          procesos y creación de soluciones inteligentes. Especializado en
          ASP.NET Core, bases de datos, integración de APIs y flujos
          automatizados con n8n.
        </p>
      </header>

      <section>
        <h3>Contacto</h3>
        <ul>
          <li>📧 ivan9ramirez6@gmail.com</li>
          <li>📱 56 2732 4942</li>
          <li>📍 Ecatepec de Morelos, Estado de México</li>
          <li>
            🔗
            <a
              href="https://www.linkedin.com/in/ivan-ignacio-ramirez-montañez-016954238"
              target="_blank"
              >LinkedIn</a
            >
          </li>
        </ul>
      </section>

      <section>
        <h3>Experiencia Profesional</h3>
        <h4>Residencia Profesional – JLCA-CVIIS Digital (2025)</h4>
        <ul>
          <li>
            Desarrollo de micrositio institucional en ASP.NET Core con Razor
            Pages
          </li>
          <li>Creación de vistas, layouts y componentes reutilizables</li>
          <li>Integración con MySQL y autenticación de usuarios</li>
          <li>Diseño responsive y buenas prácticas MVC</li>
        </ul>
      </section>

      <section>
        <h3>Formación Académica</h3>
        <p>
          <strong
            >Ingeniería en Tecnologías de la Información y
            Comunicaciones</strong
          ><br />
          Tecnológico de México – Gustavo A. Madero<br />
          2020 – Actualidad (9º semestre, especialidad en IA)
        </p>
      </section>

      <section>
        <h3>Herramientas</h3>
        <div class="tags">
          <span>HTML</span><span>CSS</span><span>JavaScript</span>
          <span>Python</span><span>ASP.NET Core</span> <span>MySQL</span
          ><span>SQL</span> <span>Android Studio</span><span>n8n</span>
        </div>
      </section>

      <section>
        <h3>Habilidades</h3>
        <ul>
          <li>Pensamiento analítico</li>
          <li>Resolución de problemas</li>
          <li>Trabajo en equipo</li>
          <li>Gestión del tiempo</li>
          <li>Aprendizaje continuo</li>
        </ul>
      </section>

      <section>
        <h3>Proyectos Destacados</h3>
        <ul>
          <li>Automatización de reportes PDF y cotizaciones con n8n</li>
          <li>Integración con Excel y generación dinámica de HTML y PDF</li>
          <li>Configuración de Webhooks, APIs y nodos personalizados</li>
        </ul>
      </section>

      <section>
        <h3>Idiomas</h3>
        <p>Español: Nativo</p>
        <p>Inglés: Básico</p>
      </section>
    </div>
  </body>
</template>
<style></style>
<script>
const boton = document.getElementById("toggle-theme");
const cuerpo = document.body;

boton.addEventListener("click", () => {
  // Alternar la clase 'dark-mode' en el body
  cuerpo.classList.toggle("dark-mode");

  // Cambiar el texto del botón según el estado
  if (cuerpo.classList.contains("dark-mode")) {
    boton.textContent = "☀️ Modo Dia";
  } else {
    boton.textContent = "🌙 Modo Noche";
  }
});
</script>

<script setup>
import { ref } from "vue";

const darkMode = ref(false);

const toggleTheme = () => {
  darkMode.value = !darkMode.value;
  document.body.classList.toggle("dark-mode");
};
</script>
