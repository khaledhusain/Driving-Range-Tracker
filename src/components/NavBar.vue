<template>
  <header class="navbar" :class="theme">
    <div class="navbar-inner">
      <!-- Left: logo -->
      <div class="nav-left">
        <img
          :src="currentLogo"
          alt="Golf Range Logo"
          class="logo"
        />
      </div>

      <!-- Center: nav links -->
      <nav class="nav-center">
        <button
          v-for="page in pages"
          :key="page.id"
          class="nav-link"
          :class="{ active: currentPage === page.id }"
          @click="$emit('change-page', page.id)"
        >
          {{ page.label }}
        </button>
      </nav>

      <!-- Right: theme toggle -->
      <div class="nav-right">
        <button class="theme-toggle" @click="$emit('toggle-theme')">
          <!-- dark theme active -> show sun to go light -->
          <span v-if="theme === 'dark'" class="icon">☀️</span>
          <!-- light theme active -> show moon to go dark -->
          <span v-else class="icon">🌙</span>
        </button>
      </div>
    </div>
  </header>
</template>

<script>
import logoDark from '../assets/golfer-dark.png'
import logoLight from '../assets/golfer-light.png'

export default {
  props: {
    theme: { type: String, required: true },        // 'dark' | 'light'
    currentPage: { type: String, required: true }   // 'bag' | 'sessions' | 'stats'
  },
  emits: ['change-page', 'toggle-theme'],
  data() {
    return {
      pages: [
        { id: 'bag', label: 'BAG' },
        { id: 'sessions', label: 'SESSIONS' },
        { id: 'stats', label: 'STATS' }
      ]
    }
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
  width: 100%;
  position: sticky;
  top: 0;
  z-index: 30;
  padding: 0.75rem 1.5rem;
  backdrop-filter: blur(10px);
}

.navbar.dark {
  background: #071423f0; /* midnight-ish */
}

.navbar.light {
  background: #f4eee5f0;
}

.navbar-inner {
  width: 100%;
  display: flex;
  align-items: center;
}

/* left: logo at top-left */
.nav-left {
  display: flex;
  align-items: center;
}

.logo {
  max-height: 56px;
  width: auto;
  display: block;
}

/* center nav */
.nav-center {
  flex: 1;
  display: flex;
  justify-content: center;
  gap: 2.25rem;
}

/* right: toggle */
.nav-right {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  min-width: 56px;
}

/* nav links */
.nav-link {
  position: relative;
  background: transparent;
  border: none;
  padding: 0;
  margin: 0;
  font-size: 0.9rem;
  font-weight: 700;             /* thicker */
  letter-spacing: 0.16em;
  text-transform: uppercase;
  cursor: pointer;
  color: inherit;
}

/* underline animation */
.nav-link::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: -6px;
  height: 2px;
  width: 100%;
  transform-origin: left center;
  transform: scaleX(0);
  transition: transform 0.18s ease-out;
}

/* underline color based on theme */
.navbar.dark .nav-link::after {
  background: #f9fafb;  /* white */
}

.navbar.light .nav-link::after {
  background: #111827;  /* almost black */
}

/* hover underline */
.nav-link:hover::after {
  transform: scaleX(1);
}

/* active link: always underlined */
.nav-link.active::after {
  transform: scaleX(1);
}

/* CLEAN TOGGLE: no circle, just emoji */
.theme-toggle {
  background: transparent;
  border: none;
  padding: 0;
  margin: 0;
  cursor: pointer;
  font-size: 1.6rem;
  line-height: 1;
}

.theme-toggle .icon {
  display: block;
}
</style>
