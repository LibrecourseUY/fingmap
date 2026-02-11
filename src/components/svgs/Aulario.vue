<script setup>
import { computed } from 'vue'
import EditableSvg from './EditableSvg.vue'

import svg0 from '../../assets/Aulario-Piso(0).svg?raw'
import svg1 from '../../assets/Aulario-Piso(1).svg?raw'
import svg2 from '../../assets/Aulario-Piso(2).svg?raw'

const props = defineProps({
  floor: {
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
  }
})

const emit = defineEmits(['room-click', 'room-hover', 'load', 'error'])

const floorMaps = {
  '0': svg0,
  '1': svg1,
  '2': svg2
}

const mapContent = computed(() => {
  const floorKey = props.floor || '0'
  return floorMaps[floorKey] || floorMaps['0']
})
</script>

<template>
  <EditableSvg
    :svg-content="mapContent"
    :theme="theme"
    :interactive="interactive"
    @room-click="$emit('room-click', $event)"
    @room-hover="$emit('room-hover', $event)"
    @load="$emit('load')"
    @error="$emit('error')"
  />
</template>
