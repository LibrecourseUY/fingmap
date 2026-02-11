<script setup>
defineProps({
  menuOpen: Boolean
})

defineEmits(['toggle-menu'])
</script>

<template>
  <header class="header">
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
        <img src="/fing.jpg" alt="FING" class="logo-icon" />
        <h1 class="logo-text">Fingmap</h1>
      </div>
    </div>
    
    <div class="header-right">
      <a 
        href="https://github.com/emiliano-gandini-outeda" 
        target="_blank" 
        rel="noopener noreferrer"
        class="github-btn"
      >
        Saber más
      </a>
    </div>
  </header>
</template>

<style scoped>
.header {
  background: linear-gradient(135deg, var(--navy-dark) 0%, var(--navy) 100%);
  color: var(--white);
  padding: 0.75rem 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: var(--shadow);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.logo-icon {
  width: 36px;
  height: 36px;
  object-fit: contain;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.1);
  padding: 2px;
}

.logo-text {
  font-size: 1.25rem;
  font-weight: 700;
  letter-spacing: -0.5px;
}

.hamburger {
  display: flex;
  flex-direction: column;
  gap: 5px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  z-index: 101;
  border-radius: 8px;
  transition: var(--transition);
}

.hamburger:hover {
  background: rgba(255, 255, 255, 0.1);
}

.hamburger span {
  width: 24px;
  height: 2px;
  background: var(--white);
  border-radius: 2px;
  transition: var(--transition);
}

.hamburger.active span:nth-child(1) {
  transform: rotate(45deg) translate(5px, 5px);
}

.hamburger.active span:nth-child(2) {
  opacity: 0;
  transform: scaleX(0);
}

.hamburger.active span:nth-child(3) {
  transform: rotate(-45deg) translate(5px, -5px);
}

.header-right {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.github-btn {
  background: rgba(255, 255, 255, 0.15);
  color: var(--white);
  padding: 0.5rem 1rem;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 600;
  font-size: 0.875rem;
  transition: var(--transition);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.github-btn:hover {
  background: rgba(255, 255, 255, 0.25);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

@media (min-width: 768px) {
  .logo-icon {
    width: 42px;
    height: 42px;
  }
  
  .logo-text {
    font-size: 1.5rem;
  }
  
  .github-btn {
    padding: 0.6rem 1.25rem;
    font-size: 1rem;
  }
}
</style>
