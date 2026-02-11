<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const props = defineProps({
  src: {
    type: String,
    required: true
  },
  theme: {
    type: String,
    default: 'dark'
  },
  interactive: {
    type: Boolean,
    default: true
  },
  showLabels: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['room-click', 'room-hover', 'load', 'error'])

const mapContainer = ref(null)
const svgContent = ref('')
const loading = ref(true)
const error = ref(null)
const parsedSvg = ref(null)

const themes = {
  dark: {
    background: '#2d2d2d',
    fill: '#3a3a3a',
    stroke: '#4a4a4a',
    text: '#ffffff',
    hover: '#4a6fa5'
  },
  light: {
    background: '#f5f5f5',
    fill: '#ffffff',
    stroke: '#cccccc',
    text: '#333333',
    hover: '#6fa5ff'
  }
}

const currentTheme = computed(() => themes[props.theme] || themes.dark)

const loadMap = async () => {
  if (!props.src) return
  
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(props.src)
    if (!response.ok) {
      throw new Error('Error al cargar el mapa')
    }
    
    const text = await response.text()
    svgContent.value = text
    
    await new Promise(resolve => setTimeout(resolve, 100))
    
    parseAndSetupSvg()
    emit('load')
    loading.value = false
  } catch (err) {
    error.value = err.message
    emit('error', err)
    loading.value = false
  }
}

const parseAndSetupSvg = () => {
  if (!mapContainer.value) return
  
  const svg = mapContainer.value.querySelector('svg')
  if (!svg) return
  
  parsedSvg.value = svg
  
  applyTheme(svg)
  
  if (props.interactive) {
    setupInteractions(svg)
  }
}

const applyTheme = (svg) => {
  const theme = currentTheme.value
  
  svg.style.background = theme.background
  svg.style.backgroundColor = theme.background
  
  svg.querySelectorAll('*').forEach(el => {
    const tagName = el.tagName.toLowerCase()
    
    if (['path', 'rect', 'polygon', 'circle', 'ellipse', 'g'].includes(tagName)) {
      const fill = el.getAttribute('fill') || ''
      const stroke = el.getAttribute('stroke') || ''
      
      if (!fill || fill === 'none' || fill === '#000' || fill === '#000000') {
        el.setAttribute('fill', theme.fill)
        el.style.fill = theme.fill
      }
      
      if (stroke && stroke !== 'none') {
        el.setAttribute('stroke', theme.stroke)
        el.style.stroke = theme.stroke
      }
    }
    
    if (tagName === 'text' || tagName === 'tspan') {
      el.setAttribute('fill', theme.text)
      el.style.fill = theme.text
      el.style.opacity = props.showLabels ? 1 : 0
    }
  })
}

const isCommonElement = (value) => {
  const commonValues = [
    'Escaleras', 'Ascensor', 'ENTRADA', 'Entrada', 'Baño', 'Baños',
    'Escalera', 'Ascensores', 'Salida', 'SERVICIO', 'Servicio',
    'Kiosco', 'Cafetería', 'Cafeteria', 'Biblioteca', 'Secretaría',
    'Secretaria', 'Aseo', 'Aseos', 'WC', 'wc', 'WC ', ' wc'
  ]
  return commonValues.some(cv => value.toLowerCase().includes(cv.toLowerCase()))
}

const setupInteractions = (svg) => {
  svg.querySelectorAll('*').forEach(el => {
    const tagName = el.tagName.toLowerCase()
    const value = el.getAttribute('value') || el.textContent?.trim() || ''
    
    if (['path', 'rect', 'polygon', 'circle', 'ellipse', 'g'].includes(tagName)) {
      if (value && value.length > 1 && !isCommonElement(value)) {
        el.style.cursor = 'pointer'
        el.style.transition = 'all 0.2s ease'
        
        const originalFill = el.getAttribute('fill') || currentTheme.value.fill
        
        el.addEventListener('click', (e) => {
          e.stopPropagation()
          emit('room-click', { element: el, value, originalEvent: e })
        })
        
        el.addEventListener('mouseenter', () => {
          el.setAttribute('fill', currentTheme.value.hover)
          el.style.fill = currentTheme.value.hover
          emit('room-hover', { element: el, value })
        })
        
        el.addEventListener('mouseleave', () => {
          el.setAttribute('fill', originalFill)
          el.style.fill = originalFill
          emit('room-hover', { element: el, value: null })
        })
      }
    }
  })
}

watch(() => props.src, () => {
  if (props.src) {
    loadMap()
  }
})

onMounted(() => {
  loadMap()
})
</script>

<template>
  <div 
    class="editable-svg" 
    ref="mapContainer"
  >
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>Cargando mapa...</p>
    </div>
    
    <div v-else-if="error" class="error">
      <p>{{ error }}</p>
      <button @click="loadMap">Reintentar</button>
    </div>
    
    <div 
      v-else 
      class="svg-container"
      v-html="svgContent"
    ></div>
  </div>
</template>

<style scoped>
.editable-svg {
  width: 100%;
  height: 100%;
  position: relative;
}

.loading,
.error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  text-align: center;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #4a4a4a;
  border-top-color: var(--navy, #003366);
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 1rem;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error button {
  margin-top: 1rem;
  padding: 0.5rem 1.5rem;
  background: var(--navy, #003366);
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.svg-container {
  width: 100%;
  height: 100%;
}

:deep(.svg-container svg) {
  width: 100%;
  height: 100%;
  display: block;
}

:deep(.svg-container svg text),
:deep(.svg-container svg tspan) {
  transition: opacity 0.3s ease;
}
</style>
