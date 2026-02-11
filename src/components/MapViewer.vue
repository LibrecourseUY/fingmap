<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import { EdificioPrincipal, Aulario } from './svgs'

const props = defineProps({
  mapSrc: String,
  building: String,
  floor: String
})

const emit = defineEmits(['room-click'])

const pan = ref({ x: 0, y: 0 })
const isDragging = ref(false)
const startPos = ref({ x: 0, y: 0 })
const lastPan = ref({ x: 0, y: 0 })
const containerRef = ref(null)
const wrapperRef = ref(null)
const svgReady = ref(false)

let lastTapTime = 0
let lastTapPos = { x: 0, y: 0 }
let startScale = ref(1)
let initialDistance = ref(0)

const ComponentMap = computed(() => {
  return props.building === 'aulario' ? Aulario : EdificioPrincipal
})

const centerMap = () => {
  if (!containerRef.value || !wrapperRef.value) return

  const container = containerRef.value
  const svg = wrapperRef.value.querySelector('.editable-svg .svg-container svg')

  if (!svg || !svgReady.value) {
    setTimeout(centerMap, 150)
    return
  }

  const containerRect = container.getBoundingClientRect()
  const svgRect = svg.getBoundingClientRect()

  if (svgRect.width > 0 && svgRect.height > 0) {
    const containerCenterX = containerRect.width / 2
    const containerCenterY = containerRect.height / 2
    const svgCenterX = svgRect.left + svgRect.width / 2
    const svgCenterY = svgRect.top + svgRect.height / 2

    pan.value = {
      x: containerCenterX - svgCenterX,
      y: containerCenterY - svgCenterY
    }

    svgReady.value = true
  } else {
    setTimeout(centerMap, 100)
  }
}

const handleRoomClick = ({ value }) => {
  emit('room-click', value)
}

const handleLoad = () => {
  console.log('Mapa cargado correctamente')
  svgReady.value = true
  setTimeout(centerMap, 80)
}

const handleError = (error) => {
  console.error('Error al cargar mapa:', error)
}

const getTouchDistance = (touch1, touch2) => {
  const dx = touch1.clientX - touch2.clientX
  const dy = touch1.clientY - touch2.clientY
  return Math.sqrt(dx * dx + dy * dy)
}

const handleTouchStart = (e) => {
  if (e.touches.length === 1) {
    lastTapTime = Date.now()
    lastTapPos = { x: e.touches[0].clientX, y: e.touches[0].clientY }

    isDragging.value = true
    startPos.value = { x: e.touches[0].clientX, y: e.touches[0].clientY }
    lastPan.value = { ...pan.value }
  } else if (e.touches.length === 2) {
    const dist = getTouchDistance(e.touches[0], e.touches[1])
    startScale.value = 1
    initialDistance.value = dist
  }
}

const handleTouchMove = (e) => {
  e.preventDefault()

  if (e.touches.length === 1 && isDragging.value) {
    const deltaX = e.touches[0].clientX - startPos.value.x
    const deltaY = e.touches[0].clientY - startPos.value.y
    pan.value = {
      x: lastPan.value.x + deltaX,
      y: lastPan.value.y + deltaY
    }
  }
}

const handleTouchEnd = () => {
  isDragging.value = false
}

const handleMouseDown = (e) => {
  if (e.button === 0) {
    isDragging.value = true
    startPos.value = { x: e.clientX, y: e.clientY }
    lastPan.value = { ...pan.value }
  }
}

const handleMouseMove = (e) => {
  if (isDragging.value) {
    const deltaX = e.clientX - startPos.value.x
    const deltaY = e.clientY - startPos.value.y
    pan.value = {
      x: lastPan.value.x + deltaX,
      y: lastPan.value.y + deltaY
    }
  }
}

const handleMouseUp = () => {
  isDragging.value = false
}

watch(() => props.mapSrc, () => {
  svgReady.value = false
  pan.value = { x: 0, y: 0 }
})

onMounted(() => {
  document.addEventListener('mouseup', handleMouseUp)
  document.addEventListener('touchend', handleTouchEnd)
})

onUnmounted(() => {
  document.removeEventListener('mouseup', handleMouseUp)
  document.removeEventListener('touchend', handleTouchEnd)
})

defineExpose({
  centerMap,
  reset: () => {
    pan.value = { x: 0, y: 0 }
  }
})
</script>

<template>
  <div 
    class="map-viewer" 
    ref="containerRef"
  >
    <div
      class="map-container"
      :class="{ 'is-dragging': isDragging }"
      @mousedown="handleMouseDown"
      @mousemove="handleMouseMove"
      @mouseup="handleMouseUp"
      @mouseleave="handleMouseUp"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
      @touchend="handleTouchEnd"
    >
      <div
        ref="wrapperRef"
        class="map-svg-wrapper"
        :style="{
          transform: `translate(${pan.x}px, ${pan.y}px)`
        }"
      >
        <ComponentMap
          v-if="floor"
          :floor="floor"
          theme="dark"
          :interactive="true"
          @room-click="handleRoomClick"
          @load="handleLoad"
          @error="handleError"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.map-viewer {
  position: fixed;
  top: 100px;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: calc(100vh - 100px);
  background: #1a1a1a;
  z-index: 1;
}

.map-container {
  width: 100%;
  height: 100%;
  overflow: hidden;
  cursor: grab;
  touch-action: none;
  background: #2d2d2d;
  display: flex;
  align-items: flex-start;
  justify-content: center;
}

.map-container.is-dragging {
  cursor: grabbing;
}

.map-svg-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.05s ease-out;
}

.map-svg-wrapper :deep(.editable-svg) {
  width: auto;
  height: auto;
  display: block;
}

.map-svg-wrapper :deep(.editable-svg .svg-container) {
  display: block;
  width: auto;
  height: auto;
}

.map-svg-wrapper :deep(.editable-svg .svg-container svg) {
  width: auto;
  height: auto;
  max-width: none;
  display: block;
}

@media (min-width: 768px) {
  .map-viewer {
    top: 120px;
    height: calc(100vh - 120px);
  }
}
</style>
