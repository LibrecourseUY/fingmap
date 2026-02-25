<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import MiniSearch from 'minisearch'

const props = defineProps({
  menuOpen: Boolean
})

const emit = defineEmits(['toggle-menu', 'zoom-reset', 'select-room'])

const searchQuery = ref('')
const searchResults = ref([])
const showResults = ref(false)
const miniSearch = ref(null)
const rooms = ref([])
const loading = ref(true)
const searchInput = ref(null)

const roomsUrl = computed(() => {
  const base = import.meta.env.BASE_URL
  return base === '/' ? '/rooms.json' : `${base}/rooms.json`
})

const loadRooms = async () => {
  try {
    const response = await fetch(roomsUrl.value)
    if (!response.ok) throw new Error(`HTTP ${response.status}`)
    rooms.value = await response.json()
    initializeSearch()
  } catch (err) {
    console.error('Error loading rooms:', err)
    rooms.value = []
  } finally {
    loading.value = false
  }
}

const initializeSearch = () => {
  if (rooms.value.length === 0) return

  miniSearch.value = new MiniSearch({
    fields: ['room', 'buildingName', 'floor'],
    storeFields: ['id', 'room', 'building', 'buildingName', 'floor', 'mapFile'],
    searchOptions: {
      boost: { room: 3, floor: 1 },
      fuzzy: 0.3,
      prefix: true
    }
  })

  miniSearch.value.addAll(rooms.value)
}

const highlightMatch = (text, query) => {
  if (!query.trim() || !text) return text
  const regex = new RegExp(`(${query.trim()})`, 'gi')
  return text.replace(regex, '<mark>$1</mark>')
}

const performSearch = () => {
  if (!searchQuery.value.trim() || !miniSearch.value) {
    searchResults.value = []
    return
  }

  const results = miniSearch.value.search(searchQuery.value, {
    fuzzy: 0.3,
    prefix: true,
    boost: { floor: 1 }
  })

  searchResults.value = results.slice(0, 8).map(r => ({
    ...r,
    highlightedRoom: highlightMatch(r.room, searchQuery.value)
  }))
  showResults.value = results.length > 0
}

const selectRoom = (room) => {
  emit('select-room', {
    id: room.id,
    room: room.room,
    building: room.building,
    floor: room.floor,
    buildingName: room.buildingName,
    mapFile: room.mapFile
  })
  showResults.value = false
  searchQuery.value = ''
}

const clearSearch = () => {
  searchQuery.value = ''
  searchResults.value = []
  showResults.value = false
}

watch(searchQuery, () => {
  performSearch()
})

watch(() => props.menuOpen, (isOpen) => {
  if (isOpen) {
    showResults.value = false
    searchQuery.value = ''
  }
})

onMounted(() => {
  loadRooms()
})
</script>

<template>
  <header class="header">
    <div v-if="!menuOpen" class="header-top">
      <div class="header-left">
        <button 
          class="hamburger" 
          @click="$emit('toggle-menu')"
          :class="{ active: menuOpen }"
          aria-label="Menú"
        >
          <span></span>
          <span></span>
          <span></span>
        </button>
        <div class="logo">
          <img src="/mapLogo.svg" alt="MapLogo" class="logo-icon" />
          <h1 class="logo-text">Fingmap</h1>
        </div>
      </div>
      
      <div class="header-right">
        <button class="reset-btn" @click="$emit('zoom-reset')" title="Centrar mapa">
          <svg viewBox="0 0 24 24" width="20" height="20">
            <path fill="currentColor" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
          </svg>
        </button>
      </div>
    </div>
    
    <div v-if="!menuOpen" class="search-bar">
      <div v-if="loading" class="search-loading">
        <div class="spinner"></div>
        <span>Cargando...</span>
      </div>
      
      <div v-else class="search-input-wrapper">
        <svg class="search-icon" viewBox="0 0 24 24" width="20" height="20">
          <path fill="currentColor" d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
        </svg>
        
        <input
          ref="searchInput"
          v-model="searchQuery"
          type="text"
          placeholder="Buscar sala..."
          class="search-input"
          @focus="showResults = true"
        />
        
        <button 
          v-if="searchQuery" 
          class="clear-btn"
          @click="clearSearch"
          aria-label="Limpiar"
        >
          <svg viewBox="0 0 24 24" width="18" height="18">
            <path fill="currentColor" d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>
          </svg>
        </button>
      </div>
      
      <Transition name="slide-down">
        <div v-if="showResults && searchResults.length > 0" class="results-dropdown">
          <ul class="results-list">
            <li 
              v-for="(result, index) in searchResults" 
              :key="result.id + '-' + index"
              class="result-item"
              @click="selectRoom(result)"
            >
              <div class="result-icon">
                <svg viewBox="0 0 24 24" width="18" height="18">
                  <path fill="currentColor" d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/>
                </svg>
              </div>
              <div class="result-info">
                <span class="result-name" v-html="result.highlightedRoom || result.room"></span>
                <span class="result-location">{{ result.buildingName }} - P{{ result.floor }}</span>
              </div>
            </li>
          </ul>
        </div>
      </Transition>
      
      <Transition name="slide-down">
        <div v-if="showResults && searchQuery && searchResults.length === 0 && !loading" class="results-dropdown no-results">
          <p>No se encontraron salas</p>
        </div>
      </Transition>
    </div>
  </header>
</template>

<style scoped>
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.3);
}

.header-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.6rem 1rem;
  height: 56px;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.logo-icon {
  width: 32px;
  height: 32px;
  object-fit: contain;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.1);
  padding: 2px;
}

.logo-text {
  font-size: 1.1rem;
  font-weight: 700;
  color: var(--white);
  letter-spacing: -0.5px;
}

.reset-btn {
  width: 38px;
  height: 38px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.15);
  border: none;
  border-radius: 8px;
  cursor: pointer;
  color: var(--white);
  transition: all 0.2s ease;
}

.reset-btn:hover {
  background: rgba(255, 255, 255, 0.25);
}

.reset-btn:active {
  transform: scale(0.92);
}

.hamburger {
  display: flex;
  flex-direction: column;
  gap: 4px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  z-index: 1001;
  border-radius: 8px;
  transition: var(--transition);
}

.hamburger:hover {
  background: rgba(255, 255, 255, 0.1);
}

.hamburger span {
  width: 22px;
  height: 2px;
  background: var(--white);
  border-radius: 2px;
  transition: var(--transition);
}

.hamburger.active span:nth-child(1) {
  transform: rotate(45deg) translate(4px, 4px);
}

.hamburger.active span:nth-child(2) {
  opacity: 0;
  transform: scaleX(0);
}

.hamburger.active span:nth-child(3) {
  transform: rotate(-45deg) translate(4px, -4px);
}

.search-bar {
  position: relative;
  padding: 0.5rem 1rem 0.75rem;
  background: rgba(0, 0, 0, 0.2);
}

.search-loading {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1rem;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
}

.spinner {
  width: 18px;
  height: 18px;
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-top-color: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.search-input-wrapper {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 10px;
  padding: 0.5rem 0.75rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: all 0.2s ease;
}

.search-input-wrapper:focus-within {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.3);
}

.search-icon {
  color: rgba(255, 255, 255, 0.6);
  flex-shrink: 0;
}

.search-input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 0.95rem;
  background: transparent;
  color: var(--white);
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

.clear-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: rgba(255, 255, 255, 0.7);
  width: 26px;
  height: 26px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.clear-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  color: var(--white);
}

.results-dropdown {
  position: absolute;
  top: 100%;
  left: 1rem;
  right: 1rem;
  background: #3a3a3a;
  border-radius: 10px;
  margin-top: 4px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.4);
  z-index: 100;
  overflow: hidden;
}

.results-list {
  list-style: none;
  max-height: 280px;
  overflow-y: auto;
}

.result-item {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 0.7rem 1rem;
  cursor: pointer;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  transition: background 0.2s ease;
}

.result-item:last-child {
  border-bottom: none;
}

.result-item:hover {
  background: rgba(0, 31, 63, 0.3);
}

.result-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  color: rgba(255, 255, 255, 0.8);
  flex-shrink: 0;
}

.result-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.1rem;
  min-width: 0;
}

.result-name {
  font-weight: 600;
  color: var(--white);
  font-size: 0.95rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.result-name :deep(mark) {
  background: rgba(255, 100, 100, 0.4);
  color: #fff;
  padding: 0 2px;
  border-radius: 2px;
}

.result-location {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.6);
}

.no-results {
  padding: 1rem;
  text-align: center;
  color: rgba(255, 255, 255, 0.6);
}

.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.2s ease;
}

.slide-down-enter-from,
.slide-down-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

@media (min-width: 768px) {
  .header-top {
    padding: 0.75rem 1.5rem;
    height: 64px;
  }

  .logo-icon {
    width: 36px;
    height: 36px;
  }

  .logo-text {
    font-size: 1.3rem;
  }

  .hamburger span {
    width: 26px;
  }

  .search-bar {
    padding: 0.6rem 1.5rem 0.8rem;
  }

  .results-dropdown {
    left: 1.5rem;
    right: 1.5rem;
  }
}
</style>
