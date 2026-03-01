<script setup>
import { ref } from 'vue'

const STORAGE_KEY = 'fingmap_terms_accepted'

const showDisclaimer = ref(!localStorage.getItem(STORAGE_KEY))

const acceptDisclaimer = () => {
  localStorage.setItem(STORAGE_KEY, 'true')
  showDisclaimer.value = false
}
</script>

<template>
  <Teleport to="body">
    <div class="disclaimer-overlay" v-if="showDisclaimer">
      <div class="disclaimer-modal">
        <h2 class="disclaimer-title">Aviso</h2>
        <div class="disclaimer-body">
          <p>LibrecourseUY es un proyecto de software open source, independiente y comunitario. Las herramientas, funcionalidades y recursos disponibles en esta plataforma —incluyendo "Fingmap"— fueron creados y mantenidos por miembros, contribuidores y colaboradores de LibrecourseUY.</p>
          <p>La herramienta "Fingmap" no está afiliada, asociada, patrocinada, avalada, autorizada ni respaldada por ninguna institución educativa u organización, incluyendo —sin limitarse a— la Universidad de la República (UdelaR) y la Facultad de Ingeniería (FING), ni por cátedras, docentes, unidades académicas o dependencias administrativas.</p>
          <p>Cualquier referencia a nombres, siglas, materias, planes de estudio, marcas o instituciones se utiliza únicamente con fines descriptivos e informativos, y no implica relación institucional, aprobación ni carácter oficial.</p>
          <p>El software y el contenido asociado se proporcionan "tal cual" (AS IS), sin garantías de ningún tipo. El uso del proyecto es bajo su propia responsabilidad.</p>
          <p>Al continuar, usted reconoce y acepta estos términos, y que cualquier interpretación de oficialidad es incorrecta.</p>
        </div>
        <button class="disclaimer-accept-btn" @click="acceptDisclaimer">Acepto los términos</button>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.disclaimer-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  animation: fadeIn 0.3s ease;
}

.disclaimer-modal {
  background-color: #fff0b3;
  border: 2px solid #ffd000;
  border-radius: 16px;
  padding: 32px;
  width: 90%;
  max-width: 600px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  animation: slideUp 0.3s ease;
}

.disclaimer-title {
  font-size: 20px;
  font-weight: 700;
  color: #1a1a1a;
  text-align: center;
  margin-bottom: 20px;
}

.disclaimer-body {
  flex: 1;
  overflow-y: auto;
  font-size: 14px;
  color: #2d2d2d;
  line-height: 1.7;
  margin-bottom: 24px;
  padding-right: 8px;
}

.disclaimer-body p {
  margin-bottom: 14px;
}

.disclaimer-accept-btn {
  width: 100%;
  padding: 14px;
  margin-top: 12px;
  background-color: #ffffff;
  color: #1a1a1a;
  border: 1px solid #1a1a1a;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  letter-spacing: 0.5px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.disclaimer-accept-btn:hover {
  background-color: #f0f0f0;
}
</style>
