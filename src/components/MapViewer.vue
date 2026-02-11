<script setup>
import { ref, watch, onMounted, nextTick } from 'vue'

const props = defineProps({
  mapSrc: String,
  highlightedRoom: String,
  building: String,
  floor: String
})

const emit = defineEmits(['room-click'])

const mapContainer = ref(null)
const svgWrapper = ref(null)
const svgContent = ref('')
const loading = ref(true)
const error = ref(null)
const scale = ref(1)
const isDragging = ref(false)
const dragStart = ref({ x: 0, y: 0 })
const pan = ref({ x: 0, y: 0 })

const loadMap = async () => {
  if (!props.mapSrc) return
  
  loading.value = true
  error.value = null
  scale.value = 1
  pan.value = { x: 0, y: 0 }
  
  try {
    const response = await fetch(props.mapSrc)
    if (!response.ok) {
      throw new Error('Error al cargar el mapa')
    }
    
    const text = await response.text()
    svgContent.value = text
    
    await nextTick()
    
    setupSvgStyles()
    setupClickHandlers()
    
    await nextTick()
    centerMap()
    
    if (props.highlightedRoom) {
      findAndHighlightRoom()
    }
    
    loading.value = false
  } catch (err) {
    error.value = err.message
    loading.value = false
  }
}

const setupSvgStyles = () => {
  const svg = mapContainer.value?.querySelector('svg')
  if (!svg) return
  
  svg.style.background = '#2d2d2d'
  svg.style.border = 'none'
  
  svg.querySelectorAll('*').forEach(el => {
    const tagName = el.tagName.toLowerCase()
    
    if (['path', 'rect', 'polygon', 'circle', 'ellipse', 'g'].includes(tagName)) {
      const fill = el.getAttribute('fill') || ''
      const stroke = el.getAttribute('stroke') || ''
      
      if (!fill || fill === 'none' || fill === '#000' || fill === '#000000') {
        el.setAttribute('fill', '#2d2d2d')
        el.style.fill = '#2d2d2d'
      }
      
      if (stroke && stroke !== 'none') {
        el.setAttribute('stroke', 'none')
        el.style.stroke = 'none'
      }
      
      el.style.strokeWidth = '0'
    }
  })
}

const findAndHighlightRoom = async () => {
  await nextTick()
  
  if (!props.highlightedRoom || !mapContainer.value) {
    return
  }
  
  const svg = mapContainer.value.querySelector('svg')
  if (!svg) return
  
  svg.querySelectorAll('.room-highlight').forEach(el => el.remove())
  
  const searchRoom = props.highlightedRoom.trim().toLowerCase()
  
  let roomElement = null
  let bestMatch = null
  let bestScore = 0
  
  const allElements = svg.querySelectorAll('*')
  for (const el of allElements) {
    const value = (el.getAttribute('value') || '').trim().toLowerCase()
    const textContent = (el.textContent || '').trim().toLowerCase()
    
    if (!value && !textContent) continue
    
    let score = 0
    if (value === searchRoom) {
      score = 100
    } else if (value.includes(searchRoom)) {
      score = 80
    } else if (textContent === searchRoom) {
      score = 90
    } else if (textContent.includes(searchRoom)) {
      score = 70
    }
    
    if (score > bestScore) {
      bestScore = score
      bestMatch = el
    }
  }
  
  roomElement = bestMatch
  
  if (roomElement && bestScore > 0) {
    try {
      const bbox = roomElement.getBBox()
      
      const highlightRect = document.createElementNS('http://www.w3.org/2000/svg', 'rect')
      highlightRect.setAttribute('x', bbox.x - 5)
      highlightRect.setAttribute('y', bbox.y - 5)
      highlightRect.setAttribute('width', bbox.width + 10)
      highlightRect.setAttribute('height', bbox.height + 10)
      highlightRect.setAttribute('class', 'room-highlight')
      highlightRect.setAttribute('fill', 'none')
      highlightRect.setAttribute('stroke', '#ff0000')
      highlightRect.setAttribute('stroke-width', '3')
      highlightRect.setAttribute('stroke-dasharray', '10, 5')
      highlightRect.style.animation = 'svgBlink 0.6s infinite alternate'
      highlightRect.style.filter = 'drop-shadow(0 0 8px rgba(255, 0, 0, 0.7))'
      
      svg.appendChild(highlightRect)
      
      if (svg.querySelector('#svgHighlightStyle')) {
        svg.querySelector('#svgHighlightStyle').remove()
      }
      const style = document.createElementNS('http://www.w3.org/2000/svg', 'style')
      style.setAttribute('id', 'svgHighlightStyle')
      style.textContent = '@keyframes svgBlink { from { stroke: #ff0000; } to { stroke: #ff4444; } }'
      svg.insertBefore(style, svg.firstChild)
      
      setTimeout(() => {
        const elRect = roomElement.getBoundingClientRect()
        const containerRect = mapContainer.value.getBoundingClientRect()
        
        pan.value = {
          x: containerRect.width / 2 - (elRect.left - containerRect.left + elRect.width / 2),
          y: containerRect.height / 2 - (elRect.top - containerRect.top + elRect.height / 2)
        }
      }, 200)
    } catch (e) {
      console.error('Error creating highlight:', e)
    }
  }
}

const setupClickHandlers = () => {
  const svg = mapContainer.value?.querySelector('svg')
  if (!svg) return
  
  svg.querySelectorAll('*').forEach(el => {
    const tagName = el.tagName.toLowerCase()
    const value = el.getAttribute('value') || ''
    
    if (['path', 'rect', 'polygon', 'circle', 'ellipse', 'g'].includes(tagName)) {
      if (value && value.length > 1 && !['Escaleras', 'Ascensor', 'ENTRADA', 'Entrada', 'Baños'].includes(value)) {
        el.style.cursor = 'pointer'
        el.style.transition = 'all 0.2s ease'
        
        el.addEventListener('click', (e) => {
          e.stopPropagation()
          emit('room-click', value)
        })
      }
    }
  })
}

const handleWheel = (e) => {
  e.preventDefault()
  const delta = e.deltaY > 0 ? -0.1 : 0.1
  scale.value = Math.max(0.25, Math.min(3, scale.value + delta))
}

const handleMouseDown = (e) => {
  if (e.button === 0) {
    isDragging.value = true
    dragStart.value = { x: e.clientX - pan.value.x, y: e.clientY - pan.value.y }
    if (mapContainer.value) {
      mapContainer.value.style.cursor = 'grabbing'
    }
  }
}

const handleMouseMove = (e) => {
  if (isDragging.value) {
    pan.value = { x: e.clientX - dragStart.value.x, y: e.clientY - dragStart.value.y }
  }
}

const handleMouseUp = () => {
  isDragging.value = false
  if (mapContainer.value) {
    mapContainer.value.style.cursor = 'grab'
  }
}

const handleZoomIn = () => {
  scale.value = Math.min(3, scale.value + 0.25)
}

const handleZoomOut = () => {
  scale.value = Math.max(0.25, scale.value - 0.25)
}

const handleReset = () => {
  scale.value = 1
  centerMap()
}

const centerMap = () => {
  const container = mapContainer.value
  const svg = container?.querySelector('svg')
  
  if (!container || !svg) return
  
  const containerRect = container.getBoundingClientRect()
  const svgRect = svg.getBoundingClientRect()
  
  if (svgRect.width > 0 && svgRect.height > 0) {
    const scaleX = containerRect.width / svgRect.width
    const scaleY = containerRect.height / svgRect.height
    scale.value = Math.min(scaleX, scaleY, 1) * 0.9
    
    pan.value = {
      x: (containerRect.width - svgRect.width * scale.value) / 2,
      y: (containerRect.height - svgRect.height * scale.value) / 2
    }
  }
}

watch(() => [props.mapSrc, props.highlightedRoom], () => {
  if (props.mapSrc) {
    loadMap()
  } else if (props.highlightedRoom) {
    findAndHighlightRoom()
  }
})

onMounted(() => {
  loadMap()
})
</script>

<template>
  <div class="map-viewer">
    <div v-if="loading" class="map-loading">
      <div class="loading-content">
        <div class="spinner"></div>
        <div class="loading-text">
          <p>Cargando mapa</p>
          <span>{{ building === 'aulario' ? 'Aulario' : 'Edificio Principal' }} - Piso {{ floor }}</span>
        </div>
      </div>
    </div>
    
    <div v-else-if="error" class="map-error">
      <div class="error-content">
        <svg viewBox="0 0 24 24" width="64" height="64">
          <path fill="#e74c3c" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
        </svg>
        <p>{{ error }}</p>
        <button @click="loadMap">Reintentar</button>
      </div>
    </div>
    
    <div v-else class="map-content">
      <div class="map-controls">
        <div class="zoom-controls">
          <button class="zoom-btn" @click="handleZoomIn" title="Acercar">
            <svg viewBox="0 0 24 24" width="20" height="20">
              <path fill="currentColor" d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/>
            </svg>
          </button>
          <button class="zoom-btn" @click="handleZoomOut" title="Alejar">
            <svg viewBox="0 0 24 24" width="20" height="20">
              <path fill="currentColor" d="M19 13H5v-2h14v2z"/>
            </svg>
          </button>
          <button class="zoom-btn" @click="handleReset" title="Restablecer">
            <svg viewBox="0 0 24 24" width="20" height="20">
              <path fill="currentColor" d="M12 5V1L7 6l5 5V7c3.31 0 6 2.69 6 6s-2.69 6-6 6-6-2.69-6-6H4c0 4.42 3.58 8 8 8s8-3.58 8-8-3.58-8-8-8z"/>
            </svg>
          </button>
        </div>
        <div class="scale-display">{{ Math.round(scale * 100) }}%</div>
      </div>
      
      <div 
        ref="mapContainer"
        class="map-scroll-container"
        @wheel="handleWheel"
        @mousedown="handleMouseDown"
        @mousemove="handleMouseMove"
        @mouseup="handleMouseUp"
        @mouseleave="handleMouseUp"
      >
        <div 
          ref="svgWrapper"
          class="map-svg-wrapper"
          :style="{ 
            transform: `translate(${pan.x}px, ${pan.y}px) scale(${scale})`,
            transformOrigin: '0 0'
          }"
        >
          <div v-html="svgContent" class="svg-content"></div>
        </div>
      </div>
      
      <div v-if="highlightedRoom" class="highlight-banner">
        <svg viewBox="0 0 24 24" width="18" height="18">
          <path fill="currentColor" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
        </svg>
        <span>Sala encontrada: <strong>{{ highlightedRoom }}</strong></span>
        <button class="clear-highlight" @click="$emit('room-click', null)">✕</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.map-viewer {
  background: #3a3a3a;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: var(--shadow);
  position: relative;
}

.map-content {
  display: flex;
  flex-direction: column;
}

.map-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  background: linear-gradient(135deg, #1a1a1a 0%, #2d2d2d 100%);
  border-bottom: 1px solid #4a4a4a;
}

.zoom-controls {
  display: flex;
  gap: 0.5rem;
}

.zoom-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  background: #3a3a3a;
  border: 1px solid #4a4a4a;
  border-radius: 8px;
  cursor: pointer;
  color: var(--white);
  transition: var(--transition);
}

.zoom-btn:hover {
  background: var(--navy);
  color: var(--white);
  border-color: var(--navy);
}

.scale-display {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-muted);
  background: #3a3a3a;
  padding: 0.4rem 0.75rem;
  border-radius: 6px;
  border: 1px solid #4a4a4a;
}

.map-scroll-container {
  width: 100%;
  height: 60vh;
  min-height: 400px;
  overflow: auto;
  background: #2d2d2d;
  cursor: grab;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.map-scroll-container::-webkit-scrollbar {
  width: 12px;
  height: 12px;
}

.map-scroll-container::-webkit-scrollbar-track {
  background: #1a1a1a;
  border-radius: 6px;
}

.map-scroll-container::-webkit-scrollbar-thumb {
  background: #4a4a4a;
  border-radius: 6px;
  border: 2px solid #2d2d2d;
}

.map-scroll-container::-webkit-scrollbar-thumb:hover {
  background: #5a5a5a;
}

.map-svg-wrapper {
  padding: 2rem;
  transition: transform 0.1s ease-out;
  min-width: min-content;
  position: relative;
}

.map-svg-wrapper .svg-content {
  position: relative;
}

.map-svg-wrapper :deep(svg) {
  width: auto;
  height: auto;
  max-width: none;
  display: block;
  background: #2d2d2d !important;
  border: none !important;
  box-shadow: none !important;
}

.map-svg-wrapper :deep(svg text) {
  font-family: 'Segoe UI', Arial, sans-serif;
  user-select: none;
}

.map-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 60vh;
  min-height: 400px;
}

.loading-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  color: var(--text-muted);
}

.spinner {
  width: 48px;
  height: 48px;
  border: 3px solid #4a4a4a;
  border-top-color: var(--navy);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-text {
  text-align: center;
}

.loading-text p {
  font-weight: 600;
  color: var(--white);
}

.loading-text span {
  font-size: 0.9rem;
  color: #b0b0b0;
}

.map-error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 60vh;
  min-height: 400px;
  padding: 2rem;
  text-align: center;
}

.error-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.error-content p {
  color: #e74c3c;
  font-weight: 600;
}

.error-content button {
  padding: 0.6rem 1.5rem;
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  color: var(--white);
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: var(--transition);
}

.error-content button:hover {
  box-shadow: var(--shadow-hover);
  transform: translateY(-2px);
}

.highlight-banner {
  position: sticky;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.875rem 1.25rem;
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  color: var(--white);
  box-shadow: 0 -4px 20px rgba(0, 31, 63, 0.2);
}

.highlight-banner svg {
  flex-shrink: 0;
  color: #2ecc71;
}

.highlight-banner span {
  flex: 1;
  font-size: 0.95rem;
}

.highlight-banner strong {
  font-weight: 700;
}

.clear-highlight {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: var(--white);
  width: 28px;
  height: 28px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  transition: var(--transition);
  flex-shrink: 0;
}

.clear-highlight:hover {
  background: rgba(255, 255, 255, 0.3);
}

@media (min-width: 768px) {
  .map-scroll-container {
    height: 65vh;
  }
}

@media (min-width: 1024px) {
  .map-scroll-container {
    height: 70vh;
  }
}
</style>
