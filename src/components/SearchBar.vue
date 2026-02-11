<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import MiniSearch from 'minisearch'

const props = defineProps({
  buildings: Array,
  currentBuilding: Object,
  currentFloor: String
})

const emit = defineEmits(['select-room'])

const searchQuery = ref('')
const searchResults = ref([])
const showResults = ref(false)
const miniSearch = ref(null)
const rooms = ref([])
const loading = ref(true)

const roomsUrl = computed(() => {
  const base = import.meta.env.BASE_URL
  return base === '/' ? '/rooms.json' : `${base}/rooms.json`
})

const loadRooms = async () => {
  try {
    console.log('Loading rooms from:', roomsUrl.value)
    const response = await fetch(roomsUrl.value)
    if (!response.ok) throw new Error(`HTTP ${response.status}`)
    rooms.value = await response.json()
    initializeSearch()
    console.log('Rooms loaded:', rooms.value.length)
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
  return text.replace(regex, '<mark class="highlight">$1</mark>')
}

const performSearch = () => {
  if (!searchQuery.value.trim() || !miniSearch.value) {
    searchResults.value = []
    return
  }
  
  const results = miniSearch.value.search(searchQuery.value, {
    fuzzy: 0.3,
    prefix: true,
    boost: { floor: props.currentFloor ? 2 : 1 }
  })
  
  searchResults.value = results.slice(0, 10).map(r => ({
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

onMounted(() => {
  loadRooms()
})
</script>

<template>
  <div class="search-container">
    <div v-if="loading" class="search-loading">
      <div class="spinner"></div>
      <span>Cargando salas...</span>
    </div>
    
    <div v-else class="search-wrapper">
      <svg class="search-icon" viewBox="0 0 24 24" width="22" height="22">
        <path fill="currentColor" d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
      </svg>
      
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Buscar salon..."
        class="search-input"
        @focus="showResults = true"
      />
      
      <button 
        v-if="searchQuery" 
        class="clear-btn"
        @click="clearSearch"
        aria-label="Limpiar búsqueda"
      >
        <svg viewBox="0 0 24 24" width="20" height="20">
          <path fill="currentColor" d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>
        </svg>
      </button>
    </div>
    
    <Transition name="fade">
      <div v-if="showResults && searchResults.length > 0" class="results-dropdown">
        <div class="results-header">
          <svg viewBox="0 0 24 24" width="16" height="16">
            <path fill="currentColor" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
          </svg>
          <span>{{ searchResults.length }} resultados</span>
        </div>
        <ul class="results-list">
          <li 
            v-for="(result, index) in searchResults" 
            :key="result.id + '-' + index"
            class="result-item"
            :class="{ 
              priority: currentBuilding?.id === result.building && currentFloor === result.floor 
            }"
            @click="selectRoom(result)"
          >
            <div class="result-icon">
              <svg viewBox="0 0 24 24" width="20" height="20">
                <path fill="currentColor" d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/>
              </svg>
            </div>
            <div class="result-info">
              <span class="result-name" v-html="result.highlightedRoom || result.room"></span>
              <span class="result-location">{{ result.buildingName }} - Piso {{ result.floor }}</span>
            </div>
            <span v-if="currentBuilding?.id === result.building && currentFloor === result.floor" class="priority-badge">
              Actual
            </span>
          </li>
        </ul>
      </div>
    </Transition>
    
    <Transition name="fade">
      <div v-if="showResults && searchQuery && searchResults.length === 0" class="results-dropdown no-results">
        <div class="no-results-content">
          <svg viewBox="0 0 24 24" width="48" height="48">
            <path fill="#e74c3c" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
          </svg>
          <p>No se encontraron salas</p>
          <span>Intenta con otros términos de búsqueda</span>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.search-container {
  position: relative;
  width: 100%;
  margin-bottom: 1.5rem;
}

.search-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 1rem;
  background: var(--white);
  border-radius: 12px;
  box-shadow: var(--shadow);
  color: var(--text-muted);
}

.spinner {
  width: 24px;
  height: 24px;
  border: 2px solid #4a4a4a;
  border-top-color: var(--navy);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.search-wrapper {
  display: flex;
  align-items: center;
  background: #3a3a3a;
  border-radius: 12px;
  padding: 0.75rem 1rem;
  box-shadow: var(--shadow);
  border: 2px solid #4a4a4a;
  transition: var(--transition);
}

.search-wrapper:focus-within {
  border-color: var(--navy);
  box-shadow: var(--shadow-hover);
}

.search-icon {
  color: var(--white);
  flex-shrink: 0;
}

.search-input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 1rem;
  padding: 0 0.75rem;
  background: transparent;
  color: var(--white);
}

.search-input::placeholder {
  color: #888;
}

.clear-btn:hover {
  background: var(--navy);
  color: var(--white);
}

.search-input::placeholder {
  color: var(--text-muted);
}

.clear-btn {
  background: #4a4a4a;
  border: none;
  color: var(--text-muted);
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
}

.clear-btn:hover {
  background: var(--gray);
  color: var(--text-dark);
}

.results-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: #3a3a3a;
  border-radius: 12px;
  margin-top: 0.5rem;
  box-shadow: var(--shadow-hover);
  max-height: 400px;
  overflow-y: auto;
  z-index: 50;
  border: 1px solid #4a4a4a;
}

.results-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1rem;
  background: #2d2d2d;
  color: var(--text-muted);
  font-size: 0.85rem;
  font-weight: 600;
  border-bottom: 1px solid #4a4a4a;
}

.results-list {
  list-style: none;
}

.result-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.875rem 1rem;
  cursor: pointer;
  border-bottom: 1px solid #3a3a3a;
  transition: var(--transition);
}

.result-item:last-child {
  border-bottom: none;
}

.result-item:hover {
  background: linear-gradient(90deg, rgba(0, 31, 63, 0.2) 0%, transparent 100%);
}

.result-item.priority {
  background: rgba(0, 31, 63, 0.1);
}

.result-item.priority:hover {
  background: linear-gradient(90deg, rgba(0, 31, 63, 0.25) 0%, transparent 100%);
}

.result-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  background: #4a4a4a;
  border-radius: 8px;
  color: var(--white);
  flex-shrink: 0;
}

.result-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  min-width: 0;
}

.result-name {
  font-weight: 600;
  color: var(--white);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.result-name :deep(.highlight) {
  background: linear-gradient(135deg, rgba(255, 0, 0, 0.3) 0%, rgba(200, 0, 0, 0.2) 100%);
  color: #ffffff;
  padding: 0.1rem 0.2rem;
  border-radius: 3px;
  font-weight: 700;
}

.result-location {
  font-size: 0.85rem;
  color: #b0b0b0;
}

.priority-badge {
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  color: var(--white);
  padding: 0.3rem 0.6rem;
  border-radius: 6px;
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  flex-shrink: 0;
}

.no-results {
  padding: 1.5rem;
  background: #3a3a3a;
}

.no-results-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  color: #b0b0b0;
}

.no-results-content p {
  margin-top: 0.75rem;
  font-weight: 600;
  color: var(--white);
}

.no-results-content span {
  font-size: 0.9rem;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@media (min-width: 768px) {
  .search-container {
    max-width: 700px;
    margin: 0 auto 2rem;
  }
}
</style>
