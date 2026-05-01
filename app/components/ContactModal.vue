<script setup lang="ts">
const props = defineProps<{ open: boolean }>();
const emit = defineEmits<{ close: [] }>();

const SCRIPT_URL =
  "https://script.google.com/macros/s/AKfycbyYLuTRTlTsNRjnqDvqngPuuCHPvI6Oov6kMpNcx_ETBXDQkDSZsu_NOi-sdGdRerAS/exec";

const form = reactive({
  nombre: "",
  email: "",
  telefono: "",
  servicio: "",
  mensaje: "",
});

const sending = ref(false);
const sent = ref(false);
const error = ref("");

const servicios = [
  "Análisis de Datos",
  "Ciberseguridad",
  "Machine Learning",
  "Desarrollo Web",
  "Consultoría",
  "Otro",
];

const closeModal = () => {
  emit("close");
  setTimeout(() => {
    sent.value = false;
    error.value = "";
    Object.assign(form, {
      nombre: "",
      email: "",
      telefono: "",
      servicio: "",
      mensaje: "",
    });
  }, 300);
};

const onSubmit = async () => {
  if (!form.nombre || !form.email || !form.mensaje) {
    error.value = "Por favor llena los campos requeridos.";
    return;
  }
  sending.value = true;
  error.value = "";

  try {
    // URLSearchParams envía como x-www-form-urlencoded
    // que SÍ es compatible con e.parameter de Apps Script
    const params = new URLSearchParams();
    params.append("nombre", form.nombre);
    params.append("email", form.email);
    params.append("telefono", form.telefono);
    params.append("servicio", form.servicio);
    params.append("mensaje", form.mensaje);

    await fetch(SCRIPT_URL, {
      method: "POST",
      mode: "no-cors",
      body: params,
    });

    sent.value = true;
  } catch (e) {
    error.value = "Hubo un error al enviar. Intenta de nuevo.";
  } finally {
    sending.value = false;
  }
};
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="open" class="overlay" @click.self="closeModal">
        <div class="modal glass">
          <!-- Header -->
          <div class="modal-header">
            <div class="modal-title-wrap">
              <span class="modal-icon">✉</span>
              <h2 class="modal-title">Contacto</h2>
            </div>
            <button class="close-btn" @click="closeModal">✕</button>
          </div>

          <!-- Success -->
          <div v-if="sent" class="success-state">
            <div class="success-icon">✓</div>
            <h3 class="success-title">¡Mensaje enviado!</h3>
            <p class="success-text">
              Gracias por contactarme. Te responderé pronto.
            </p>
            <button class="submit-btn" @click="closeModal">Cerrar</button>
          </div>

          <!-- Form -->
          <form v-else class="modal-form" @submit.prevent="onSubmit">
            <div class="form-row">
              <div class="form-group">
                <label class="form-label"
                  >Nombre <span class="required">*</span></label
                >
                <input
                  v-model="form.nombre"
                  class="form-input"
                  type="text"
                  placeholder="Tu nombre completo"
                />
              </div>
              <div class="form-group">
                <label class="form-label"
                  >Email <span class="required">*</span></label
                >
                <input
                  v-model="form.email"
                  class="form-input"
                  type="email"
                  placeholder="tu@email.com"
                />
              </div>
            </div>

            <div class="form-row">
              <div class="form-group">
                <label class="form-label">Teléfono</label>
                <input
                  v-model="form.telefono"
                  class="form-input"
                  type="tel"
                  placeholder="+52 55 1234 5678"
                />
              </div>
              <div class="form-group">
                <label class="form-label">Tipo de servicio</label>
                <select v-model="form.servicio" class="form-input form-select">
                  <option value="" disabled>Selecciona...</option>
                  <option v-for="s in servicios" :key="s" :value="s">
                    {{ s }}
                  </option>
                </select>
              </div>
            </div>

            <div class="form-group">
              <label class="form-label"
                >Mensaje <span class="required">*</span></label
              >
              <textarea
                v-model="form.mensaje"
                class="form-input form-textarea"
                placeholder="Cuéntame en qué puedo ayudarte..."
                rows="4"
              ></textarea>
            </div>

            <p v-if="error" class="form-error">⚠ {{ error }}</p>

            <div class="form-actions">
              <button type="button" class="cancel-btn" @click="closeModal">
                Cancelar
              </button>
              <button type="submit" class="submit-btn" :disabled="sending">
                <span v-if="sending" class="spinner-sm"></span>
                {{ sending ? "Enviando..." : "Enviar mensaje" }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(6px);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.modal {
  width: 100%;
  max-width: 560px;
  max-height: 90vh;
  overflow-y: auto;
  padding: 0;
  border-color: rgba(0, 255, 238, 0.2);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.2rem 1.5rem;
  border-bottom: 1px solid rgba(0, 255, 238, 0.1);
  background: rgba(0, 0, 0, 0.2);
}

.modal-title-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
}

.modal-icon {
  font-size: 1.2rem;
  color: #00ffee;
}

.modal-title {
  font-family: "Space Grotesk", sans-serif;
  font-size: 1.1rem;
  font-weight: 700;
  color: #fff;
  margin: 0;
}

.close-btn {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: rgba(180, 180, 220, 0.7);
  width: 32px;
  height: 32px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}
.close-btn:hover {
  background: rgba(255, 80, 80, 0.15);
  border-color: rgba(255, 80, 80, 0.4);
  color: #ff5050;
}

.modal-form {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

@media (max-width: 480px) {
  .form-row {
    grid-template-columns: 1fr;
  }
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-label {
  font-size: 0.72rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #00ffee;
  font-family: "Fira Code", monospace;
}

.required {
  color: #ff5050;
}

.form-input {
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(0, 255, 238, 0.15);
  border-radius: 8px;
  padding: 0.65rem 0.9rem;
  color: #e8e8ff;
  font-family: "Space Grotesk", sans-serif;
  font-size: 0.88rem;
  outline: none;
  transition:
    border-color 0.2s,
    box-shadow 0.2s;
  width: 100%;
}

.form-input:focus {
  border-color: rgba(0, 255, 238, 0.5);
  box-shadow: 0 0 0 3px rgba(0, 255, 238, 0.06);
}

.form-input::placeholder {
  color: rgba(160, 160, 200, 0.4);
}

.form-select {
  cursor: pointer;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' fill='%2300ffee' viewBox='0 0 16 16'%3E%3Cpath d='M7.247 11.14L2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 0 1 .753 1.659l-4.796 5.48a1 1 0 0 1-1.506 0z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 0.9rem center;
  padding-right: 2.2rem;
}

.form-select option {
  background: #0a0a1a;
  color: #e8e8ff;
}

.form-textarea {
  resize: vertical;
  min-height: 100px;
}

.form-error {
  color: #ff5050;
  font-size: 0.82rem;
  background: rgba(255, 80, 80, 0.08);
  border: 1px solid rgba(255, 80, 80, 0.2);
  padding: 0.6rem 0.9rem;
  border-radius: 8px;
}

.form-actions {
  display: flex;
  gap: 0.75rem;
  justify-content: flex-end;
  padding-top: 0.5rem;
}

.cancel-btn {
  background: transparent;
  border: 1px solid rgba(0, 255, 238, 0.2);
  color: rgba(160, 160, 200, 0.7);
  padding: 0.6rem 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  font-family: "Space Grotesk", sans-serif;
  font-size: 0.85rem;
  transition: all 0.2s;
}
.cancel-btn:hover {
  border-color: rgba(0, 255, 238, 0.4);
  color: #00ffee;
}

.submit-btn {
  background: #00ffee;
  color: #0a0a1a;
  border: none;
  padding: 0.6rem 1.4rem;
  border-radius: 8px;
  cursor: pointer;
  font-family: "Fira Code", monospace;
  font-weight: 700;
  font-size: 0.85rem;
  letter-spacing: 0.05em;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  gap: 8px;
}
.submit-btn:hover:not(:disabled) {
  background: #90cdf4;
  transform: translateY(-1px);
}
.submit-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.spinner-sm {
  width: 14px;
  height: 14px;
  border: 2px solid rgba(0, 0, 0, 0.2);
  border-top-color: #0a0a1a;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Success state */
.success-state {
  padding: 2.5rem 1.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  text-align: center;
}

.success-icon {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background: rgba(0, 255, 180, 0.1);
  border: 2px solid #00ff88;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.8rem;
  color: #00ff88;
}

.success-title {
  font-family: "Space Grotesk", sans-serif;
  font-size: 1.2rem;
  color: #fff;
  margin: 0;
}

.success-text {
  color: rgba(160, 160, 200, 0.7);
  font-size: 0.88rem;
}

/* Transitions */
.modal-enter-active,
.modal-leave-active {
  transition: all 0.25s ease;
}
.modal-enter-from,
.modal-leave-to {
  opacity: 0;
  transform: scale(0.95) translateY(10px);
}
</style>
