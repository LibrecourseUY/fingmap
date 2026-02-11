<script setup>
import { ref } from 'vue'

const props = defineProps({
  isOpen: Boolean,
  buildings: Array,
  currentBuilding: Object,
  currentFloor: String
})

const emit = defineEmits(['select-building', 'select-floor', 'close'])

const openBuilding = ref(null)

const toggleBuilding = (buildingId) => {
  if (openBuilding.value === buildingId) {
    openBuilding.value = null
  } else {
    openBuilding.value = buildingId
  }
}

const onSelectBuilding = (building) => {
  emit('select-building', building)
  openBuilding.value = building.id
}

const onSelectFloor = (floor) => {
  emit('select-floor', floor)
}
</script>

<template>
  <Transition name="slide">
    <div v-if="isOpen" class="menu-overlay" @click.self="$emit('close')">
      <nav class="menu">
        <div class="menu-header">
          <h2>Navegación</h2>
          <button class="close-btn" @click="$emit('close')" aria-label="Cerrar menú">
            <svg viewBox="0 0 24 24" width="24" height="24">
              <path fill="currentColor" d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>
            </svg>
          </button>
        </div>
        
        <ul class="building-list">
          <li 
            v-for="building in buildings" 
            :key="building.id"
            class="building-item"
          >
            <button 
              class="building-btn"
              :class="{ 
                expanded: openBuilding === building.id
              }"
              @click="onSelectBuilding(building)"
            >
              <span class="building-icon">
                <svg viewBox="0 0 24 24" width="20" height="20">
                  <path fill="currentColor" d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/>
                </svg>
              </span>
              <span class="building-name">{{ building.name }}</span>
              <span class="arrow">▼</span>
            </button>
            
            <Transition name="expand">
              <div v-if="openBuilding === building.id" class="floor-dropdown">
                <div class="floor-dropdown-header">Seleccionar piso</div>
                <ul class="floor-list">
                  <li 
                    v-for="floor in building.floors" 
                    :key="floor"
                    class="floor-item"
                  >
                    <button 
                      class="floor-btn"
                      :class="{ active: currentFloor === floor && currentBuilding?.id === building.id }"
                      @click="onSelectFloor(floor)"
                    >
                      <span class="floor-icon">
                        <svg viewBox="0 0 24 24" width="16" height="16">
                          <path fill="currentColor" d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 16H5V5h14v14z"/>
                        </svg>
                      </span>
                      Piso {{ floor }}
                    </button>
                  </li>
                </ul>
              </div>
            </Transition>
          </li>
        </ul>
        
        <div class="menu-footer">
          <div class="current-selection" v-if="currentBuilding && currentFloor">
            <span class="selection-label">Actual:</span>
            <span class="selection-value">{{ currentBuilding.name }} - Piso {{ currentFloor }}</span>
          </div>
        </div>
      </nav>
    </div>
  </Transition>
</template>

<style scoped>
.menu-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 200;
  display: flex;
  backdrop-filter: blur(4px);
}

.menu {
  width: 85%;
  max-width: 340px;
  height: 100%;
  background: linear-gradient(180deg, #3a3a3a 0%, #2d2d2d 100%);
  box-shadow: 4px 0 30px rgba(0, 0, 0, 0.4);
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.menu-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.25rem 1rem;
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  color: var(--white);
}

.menu-header h2 {
  font-size: 1.2rem;
  font-weight: 600;
}

.close-btn {
  background: rgba(255, 255, 255, 0.15);
  border: none;
  color: var(--white);
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
}

.close-btn:hover {
  background: rgba(255, 255, 255, 0.25);
}

.building-list {
  list-style: none;
  flex: 1;
  padding: 0.5rem;
}

.building-item {
  margin-bottom: 0.5rem;
}

.building-btn {
  width: 100%;
  padding: 1rem;
  background: #3a3a3a;
  border: 1px solid #4a4a4a;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  color: var(--text-dark);
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  transition: var(--transition);
}

.building-btn:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  transform: translateX(4px);
  background: #4a4a4a;
}

.building-btn.expanded {
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  color: var(--white);
  border-color: var(--navy);
}

.building-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0.9;
  color: var(--white);
}

.building-name {
  flex: 1;
  text-align: left;
  color: var(--text-dark);
}

.arrow {
  font-size: 0.65rem;
  transition: transform 0.3s ease;
  color: var(--text-muted);
}

.building-btn.expanded .arrow {
  transform: rotate(180deg);
  color: var(--white);
}

.floor-dropdown {
  background: #2d2d2d;
  border-radius: 12px;
  margin-top: 0.5rem;
  overflow: hidden;
  border: 1px solid #4a4a4a;
}

.floor-dropdown-header {
  padding: 0.75rem 1rem 0.5rem;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  color: var(--text-muted);
  font-weight: 600;
}

.floor-list {
  list-style: none;
  padding: 0.25rem 0.5rem 0.75rem;
}

.floor-btn {
  width: 100%;
  padding: 0.75rem 1rem;
  background: #3a3a3a;
  border: 1px solid #4a4a4a;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  text-align: left;
  cursor: pointer;
  font-size: 0.95rem;
  color: var(--text-dark);
  border-radius: 8px;
  transition: var(--transition);
}

.floor-btn:hover {
  background: var(--navy);
  color: var(--white);
  transform: translateX(4px);
  border-color: var(--navy);
}

.floor-btn.active {
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  color: var(--white);
  box-shadow: 0 2px 8px rgba(0, 31, 63, 0.4);
  border-color: var(--navy);
}

.floor-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0.8;
  color: var(--white);
}

.menu-footer {
  padding: 1rem;
  border-top: 1px solid #4a4a4a;
  background: #3a3a3a;
}

.current-selection {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  border-radius: 10px;
  color: var(--white);
}

.selection-label {
  font-size: 0.8rem;
  opacity: 0.8;
}

.selection-value {
  font-weight: 600;
  font-size: 0.9rem;
}

.slide-enter-active,
.slide-leave-active {
  transition: opacity 0.3s ease;
}

.slide-enter-from,
.slide-leave-to {
  opacity: 0;
}

.slide-enter-active .menu,
.slide-leave-active .menu {
  transition: transform 0.3s ease;
}

.slide-enter-from .menu,
.slide-leave-to .menu {
  transform: translateX(-100%);
}

.expand-enter-active,
.expand-leave-active {
  transition: all 0.3s ease;
  max-height: 500px;
}

.expand-enter-from,
.expand-leave-to {
  max-height: 0;
  opacity: 0;
}

@media (min-width: 768px) {
  .menu {
    width: 380px;
  }
}
</style>
