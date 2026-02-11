<script setup>
import { computed } from 'vue'
import EditableSvg from './EditableSvg.vue'

import svgNeg2 from '../../assets/Edificio-Principal-Piso(-2).svg?raw'
import svgNeg1 from '../../assets/Edificio-Principal-Piso(-1).svg?raw'
import svg0 from '../../assets/Edificio-Principal-Piso(0).svg?raw'
import svg1 from '../../assets/Edificio-Principal-Piso(1).svg?raw'
import svg2 from '../../assets/Edificio-Principal-Piso(2).svg?raw'
import svg3 from '../../assets/Edificio-Principal-Piso(3).svg?raw'
import svg4 from '../../assets/Edificio-Principal-Piso(4).svg?raw'
import svg5 from '../../assets/Edificio-Principal-Piso(5).svg?raw'
import svg6 from '../../assets/Edificio-Principal-Piso(6).svg?raw'
import svg7 from '../../assets/Edificio-Principal-Piso(7).svg?raw'

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
  '-2': svgNeg2,
  '-1': svgNeg1,
  '0': svg0,
  '1': svg1,
  '2': svg2,
  '3': svg3,
  '4': svg4,
  '5': svg5,
  '6': svg6,
  '7': svg7
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
