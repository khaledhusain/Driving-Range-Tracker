<template>
  <div :class="['app', theme]">
    <NavBar
      :theme="theme"
      :current-page="currentPage"
      @change-page="setPage"
      @toggle-theme="toggleTheme"
    />

    <main class="content">
      <!-- BAG PAGE -->
      <BagPage
        v-if="currentPage === 'bag'"
        :clubs="sortedClubs"
        @add-club="addClub"
        @delete-club="deleteClub"
      />

      <!-- SESSIONS PAGE -->
      <SessionsPage
        v-else-if="currentPage === 'sessions'"
        :clubs="sortedClubs"
        :sessions="sessions"
        @add-session="addSession"
      />

      <!-- STATS PAGE -->
      <StatsPage
        v-else
        :sessions="sessions"
        :clubs="clubs"
      />
    </main>
  </div>
</template>

<script>
import NavBar from './components/NavBar.vue'
import BagPage from './components/BagPage.vue'
import SessionsPage from './components/SessionsPage.vue'
import StatsPage from './components/StatsPage.vue'

export default {
  components: { NavBar, BagPage, SessionsPage, StatsPage },

  data() {
    return {
      theme: 'dark',          // 'dark' | 'light'
      currentPage: 'bag',     // 'bag' | 'sessions' | 'stats'

      // clubs WITH ids
      clubs: [
        { id: 1, name: 'Driver', typicalDistance: 250, notes: 'Big tee shots' },
        { id: 2, name: '7 Iron', typicalDistance: 165, notes: 'Reliable' },
        { id: 3, name: '52° Wedge', typicalDistance: 105, notes: 'Dial-in' }
      ],

      // sessions: one per visit, multiple clubs inside
      sessions: [
        {
          id: 1,
          date: '2025-11-18',
          notes: 'Evening grind',
          entries: [
            {
              id: 1,
              clubId: 1,
              ballsHit: 40,
              averageDistance: 240,
              bestDistance: 275
            },
            {
              id: 2,
              clubId: 2,
              ballsHit: 30,
              averageDistance: 165,
              bestDistance: 185
            }
          ]
        }
      ]
    }
  },

  computed: {
    // sort by distance DESC so longer clubs sit higher
    sortedClubs() {
      return [...this.clubs].sort(
        (a, b) => b.typicalDistance - a.typicalDistance
      )
    }
  },

  methods: {
    setPage(page) {
      this.currentPage = page
    },

    toggleTheme() {
      this.theme = this.theme === 'dark' ? 'light' : 'dark'
    },

    // called from BagPage modal
    addClub(payload) {
      const { name, typicalDistance, notes } = payload
      if (!name || !typicalDistance) return

      // make sure every club has a unique id
      const nextId = this.clubs.length
        ? Math.max(...this.clubs.map(c => c.id || 0)) + 1
        : 1

      this.clubs.push({
        id: nextId,
        name: name.trim(),
        typicalDistance,
        notes: notes.trim()
      })
    },

    // swipe-to-delete from BagPage
    deleteClub(id) {
      this.clubs = this.clubs.filter(c => c.id !== id)
    },

    // sessions: payload = { date, notes, entries: [{ clubId, ballsHit, averageDistance, bestDistance }] }
    addSession(payload) {
      const { date, notes, entries } = payload
      if (!date || !entries || !entries.length) return

      const nextSessionId = this.sessions.length
        ? Math.max(...this.sessions.map(s => s.id)) + 1
        : 1

      const entriesWithIds = entries.map((entry, idx) => ({
        id: idx + 1,
        ...entry
      }))

      this.sessions.push({
        id: nextSessionId,
        date,
        notes: notes.trim(),
        entries: entriesWithIds
      })
    }
  }
}
</script>

<style scoped>
.app.dark {
  /* dark mode – match midnight blue logo */
  --bg: #071423;
  --card: #0b1826;
  --text: #f9fafb;
  --muted: #9ca3af;
  --accent: #2f7a4c;
  --border: #1f2937;
}

.app.light {
  /* light mode – match cream logo background */
  --bg: #f4eee5;
  --card: #ffffff;
  --text: #111827;
  --muted: #6b7280;
  --accent: #2f7a4c;
  --border: #ddd4c7;
}

.app {
  min-height: 100vh;
  background: var(--bg);
  color: var(--text);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI',
    sans-serif;
}

.content {
  max-width: 900px;
  margin: 0 auto;
  padding: 1.5rem 1rem 2.5rem;
}
</style>

