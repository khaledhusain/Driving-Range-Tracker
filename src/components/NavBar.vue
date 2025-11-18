<template>
  <header class="navbar">
    <div class="brand">
      <img :src="currentLogo" alt="Golf logo" class="logo" />
      <span class="brand-text">Range Tracker</span>
    </div>

    <nav class="nav-links">
      <button
        class="nav-btn"
        :class="{ active: currentPage === 'bag' }"
        @click="$emit('change-page', 'bag')"
      >
        Bag
      </button>
      <button
        class="nav-btn"
        :class="{ active: currentPage === 'sessions' }"
        @click="$emit('change-page', 'sessions')"
      >
        Sessions
      </button>
      <button
        class="nav-btn"
        :class="{ active: currentPage === 'stats' }"
        @click="$emit('change-page', 'stats')"
      >
        Stats
      </button>
    </nav>

    <button class="theme-toggle" @click="$emit('toggle-theme')">
      {{ theme === 'dark' ? '☀️' : '🌙' }}
    </button>
  </header>
</template>

<script>
import logoDark from '../assets/golfer-dark.png'
import logoLight from '../assets/golfer-light.png'

export default {
  props: {
    theme: { type: String, required: true },
    currentPage: { type: String, required: true }
  },
  computed: {
    currentLogo() {
      return this.theme === 'dark' ? logoDark : logoLight
    }
  }
}
</script>

<style scoped>
.navbar {
  background: var(--bg);
  border-bottom: 1px solid var(--border);
  padding: 0.75rem 1.25rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.brand {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.logo {
  height: 42px;
}

.brand-text {
  font-weight: 600;
  font-size: 1.1rem;
}

.nav-links {
  display: flex;
  gap: 0.5rem;
}

.nav-btn {
  background: transparent;
  border: 1px solid transparent;
  padding: 0.35rem 0.9rem;
  border-radius: 999px;
  color: var(--muted);
  cursor: pointer;
}

.nav-btn.active {
  background: var(--accent);
  color: #022c22;
}

.theme-toggle {
  border-radius: 999px;
  padding: 0.3rem 0.7rem;
  border: 1px solid var(--border);
  background: transparent;
  cursor: pointer;
}
</style>
