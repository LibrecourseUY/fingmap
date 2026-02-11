<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const props = defineProps({
  svgContent: {
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

const parseAndSetupSvg = () => {
  if (!mapContainer.value || !props.svgContent) return

  mapContainer.value.innerHTML = props.svgContent

  const svg = mapContainer.value.querySelector('svg')
  if (!svg) return

  parsedSvg.value = svg

  applyTheme(svg)

  if (props.interactive) {
    setupInteractions(svg)
  }

  emit('load')
}

const applyTheme = (svg) => {
  const theme = currentTheme.value

  svg.style.background = theme.background
  svg.style.backgroundColor = theme.background
  svg.style.colorScheme = 'dark'

  svg.querySelectorAll('*').forEach(el => {
    const tagName = el.tagName.toLowerCase()
    const fill = el.getAttribute('fill') || ''
    const stroke = el.getAttribute('stroke') || ''

    if (['path', 'rect', 'polygon', 'circle', 'ellipse', 'g'].includes(tagName)) {
      if (!fill || fill === 'none' || fill === '#000' || fill === '#000000' || fill === 'rgb(0, 0, 0)' || fill === 'black') {
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
      el.style.color = theme.text
      el.setAttribute('color', theme.text)
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

watch(() => props.svgContent, () => {
  parseAndSetupSvg()
})

onMounted(() => {
  parseAndSetupSvg()
})
</script>

<template>
  <div
    class="editable-svg"
    ref="mapContainer"
  ></div>
</template>

<style scoped>
.editable-svg {
  width: 100%;
  height: 100%;
  position: relative;
}

:deep(.editable-svg svg) {
  width: 100%;
  height: 100%;
  display: block;
}

:deep(.editable-svg svg text),
:deep(.editable-svg svg tspan) {
  transition: opacity 0.3s ease;
}
</style>
