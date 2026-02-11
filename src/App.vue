<script setup>
import { ref, computed, onMounted } from 'vue'
import Header from './components/Header.vue'
import HamburgerMenu from './components/HamburgerMenu.vue'
import SearchBar from './components/SearchBar.vue'
import MapViewer from './components/MapViewer.vue'

const currentBuilding = ref(null)
const currentFloor = ref(null)
const menuOpen = ref(false)
const highlightedRoom = ref(null)

const buildings = [
  {
    id: 'edificio-principal',
    name: 'Edificio Principal',
    floors: ['-2', '-1', '0', '1', '2', '3', '4', '5', '6', '7']
  },
  {
    id: 'aulario',
    name: 'Aulario',
    floors: ['0', '1', '2']
  }
]

const currentMap = computed(() => {
  if (!currentBuilding.value || !currentFloor.value) return null
  
  const buildingPrefix = currentBuilding.value.id === 'aulario' ? 'Aulario' : 'Edificio-Principal'
  return `/images/${buildingPrefix}-Piso(${currentFloor.value}).svg`
})

const toggleMenu = () => {
  menuOpen.value = !menuOpen.value
}

const selectBuilding = (building) => {
  currentBuilding.value = building
  currentFloor.value = null
}

const selectFloor = (floor) => {
  currentFloor.value = floor
  highlightedRoom.value = null
  menuOpen.value = false
}

const handleRoomSelect = (room) => {
  if (room.building !== currentBuilding.value?.id) {
    const building = buildings.find(b => b.id === room.building)
    currentBuilding.value = building
  }
  if (room.floor !== currentFloor.value) {
    currentFloor.value = room.floor
  }
  highlightedRoom.value = room.room
}

const clearHighlight = () => {
  highlightedRoom.value = null
}

onMounted(() => {
  if (buildings.length > 0) {
    currentBuilding.value = buildings[0]
    currentFloor.value = '0'
  }
})
</script>

<template>
  <div class="app">
    <Header 
      @toggle-menu="toggleMenu"
      :menu-open="menuOpen"
    />
    
    <HamburgerMenu 
      :is-open="menuOpen"
      :buildings="buildings"
      :current-building="currentBuilding"
      :current-floor="currentFloor"
      @select-building="selectBuilding"
      @select-floor="selectFloor"
      @close="menuOpen = false"
    />
    
    <main class="main-content">
      <SearchBar 
        :buildings="buildings"
        :current-building="currentBuilding"
        :current-floor="currentFloor"
        @select-room="handleRoomSelect"
      />
      
      <MapViewer 
        v-if="currentMap"
        :map-src="currentMap"
        :highlighted-room="highlightedRoom"
        :building="currentBuilding?.id"
        :floor="currentFloor"
        @room-click="clearHighlight"
      />
      
      <div v-else class="no-map">
        <p>Selecciona un edificio y piso para ver el mapa</p>
      </div>
    </main>
  </div>
</template>

<style>
:root {
  --navy-dark: #001f3f;
  --navy: #003366;
  --navy-light: #004080;
  --accent: #003366;
  --white: #ffffff;
  --gray-dark: #2d2d2d;
  --gray: #3a3a3a;
  --gray-light: #4a4a4a;
  --text-dark: #ffffff;
  --text-muted: #b0b0b0;
  --shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  --shadow-hover: 0 8px 30px rgba(0, 0, 0, 0.4);
  --transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  scroll-behavior: smooth;
}

body {
  font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, Roboto, Oxygen, Ubuntu, sans-serif;
  background: linear-gradient(135deg, #1a1a1a 0%, #2d2d2d 50%, #1a1a1a 100%);
  color: var(--text-dark);
  line-height: 1.6;
  min-height: 100vh;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: rgba(45, 45, 45, 0.95);
}

.main-content {
  flex: 1;
  padding: 1rem;
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;
}

.no-map {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
  color: var(--text-muted);
  font-size: 1.1rem;
  background: #3a3a3a;
  border-radius: 16px;
  box-shadow: var(--shadow);
  padding: 2rem;
  border: 1px solid #4a4a4a;
}

@media (min-width: 768px) {
  .main-content {
    padding: 1.5rem;
  }
  
  .no-map {
    font-size: 1.25rem;
    padding: 3rem;
  }
}

@media (min-width: 1024px) {
  .main-content {
    padding: 2rem;
  }
}
</style>
