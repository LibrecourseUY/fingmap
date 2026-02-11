<script setup>
import { computed } from 'vue'
import EditableSvg from './EditableSvg.vue'

const props = defineProps({
  floor: {
    type: String,
    required: true
  },
  highlightedRoom: {
    type: String,
    default: null
  },
  theme: {
    type: String,
    default: 'dark'
  },
  interactive: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['room-click', 'room-hover', 'load', 'error'])

const floorMap = {
  '0': 'Aulario-Piso(0)',
  '1': 'Aulario-Piso(1)',
  '2': 'Aulario-Piso(2)'
}

const mapSrc = computed(() => {
  const floorKey = props.floor || '0'
  const filename = floorMap[floorKey] || floorMap['0']
  const base = import.meta.env.BASE_URL
  const mapPath = `/images/${filename}.svg`
  return base === '/' ? mapPath : `${base}${mapPath}`
})

const handleRoomClick = (data) => {
  emit('room-click', data)
}

const handleRoomHover = (data) => {
  emit('room-hover', data)
}
</script>

<template>
  <EditableSvg
    :src="mapSrc"
    :highlighted-room="highlightedRoom"
    :theme="theme"
    :interactive="interactive"
    @room-click="handleRoomClick"
    @room-hover="handleRoomHover"
    @load="$emit('load')"
    @error="$emit('error')"
  >
    <template #loading>
      <div class="loading-content">
        <div class="spinner"></div>
        <p>Cargando mapa</p>
        <span>Aulario - Piso {{ floor }}</span>
      </div>
    </template>
  </EditableSvg>
</template>

<style scoped>
.loading-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #4a4a4a;
  border-top-color: var(--navy, #003366);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-content p {
  font-weight: 600;
  color: var(--white, #ffffff);
}

.loading-content span {
  font-size: 0.9rem;
  color: #b0b0b0;
}
</style>
