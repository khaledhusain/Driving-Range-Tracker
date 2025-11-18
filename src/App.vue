<template>
  <div :class="['app', theme]">
    <NavBar
      :theme="theme"
      :current-page="currentPage"
      @change-page="setPage"
      @toggle-theme="toggleTheme"
    />

    <main class="content">
      <BagPage
        v-if="currentPage === 'bag'"
        :clubs="sortedClubs"
        @add-club="addClub"
      />

      <SessionsPage
        v-else-if="currentPage === 'sessions'"
        :clubs="sortedClubs"
        :sessions="sessions"
        @add-session="addSession"
      />

      <StatsPage
        v-else
        :total-sessions="totalSessions"
        :total-balls="totalBalls"
        :most-used-club-name="mostUsedClubName"
        :best-shot="bestShot"
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
      theme: 'dark',
      currentPage: 'bag',

      clubs: [
        { id: 1, name: 'Driver', typicalDistance: 250, notes: 'Big tee shots' },
        { id: 2, name: '7 Iron', typicalDistance: 165, notes: 'Reliable' },
        { id: 3, name: '52° Wedge', typicalDistance: 105, notes: 'Dial-in' }
      ],

      sessions: [
        {
          id: 101,
          date: '2025-11-14',
          clubId: 1,
          ballsHit: 60,
          averageDistance: 240,
          bestDistance: 280,
          notes: 'Solid practice'
        }
      ]
    }
  },

  computed: {
    // 👇 SORT BY TYPICAL DISTANCE DESCENDING
    sortedClubs() {
      return [...this.clubs].sort(
        (a, b) => b.typicalDistance - a.typicalDistance
      )
    },

    totalSessions() {
      return this.sessions.length
    },

    totalBalls() {
      return this.sessions.reduce((sum, s) => sum + (s.ballsHit || 0), 0)
    },

    bestShot() {
      if (!this.sessions.length) return null
      return Math.max(...this.sessions.map(s => s.bestDistance || 0))
    },

    mostUsedClubName() {
      if (!this.sessions.length) return null

      const counts = {}
      for (const s of this.sessions) {
        counts[s.clubId] = (counts[s.clubId] || 0) + 1
      }

      const topId = Number(
        Object.entries(counts).sort((a, b) => b[1] - a[1])[0][0]
      )

      const club = this.clubs.find(c => c.id === topId)
      return club ? club.name : null
    }
  },

  methods: {
    setPage(page) {
      this.currentPage = page
    },

    toggleTheme() {
      this.theme = this.theme === 'dark' ? 'light' : 'dark'
    },

    addClub(payload) {
      const { name, typicalDistance, notes } = payload
      if (!name || !typicalDistance) return

      const nextId = this.clubs.length
        ? Math.max(...this.clubs.map(c => c.id)) + 1
        : 1

      this.clubs.push({
        id: nextId,
        name: name.trim(),
        typicalDistance,
        notes: notes.trim()
      })
    },

    addSession(payload) {
      const {
        date,
        clubId,
        ballsHit,
        averageDistance,
        bestDistance,
        notes
      } = payload

      if (!date || !clubId || !ballsHit || !averageDistance || !bestDistance) {
        return
      }

      const nextId = this.sessions.length
        ? Math.max(...this.sessions.map(s => s.id)) + 1
        : 1

      this.sessions.push({
        id: nextId,
        date,
        clubId,
        ballsHit,
        averageDistance,
        bestDistance,
        notes: notes.trim()
      })
    }
  }
}
</script>

<style scoped>
/* THEME VARIABLES */
.app.dark {
  --bg: #071423; /* midnight blue family */
  --card: #111827;
  --text: #f9fafb;
  --muted: #9ca3af;
  --accent: #2f7a4c;
  --border: #273549;
}

.app.light {
  --bg: #f4eee5;
  --card: #ffffff;
  --text: #111827;
  --muted: #6b7280;
  --accent: #2f7a4c;
  --border: #e1ded7;
}

.app {
  min-height: 100vh;
  background: var(--bg);
  color: var(--text);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI',
    sans-serif;
  transition: background-color 0.2s ease, color 0.2s ease;
}

.content {
  max-width: 900px;
  margin: 0 auto;
  padding: 1.5rem 1rem 2.5rem;
}

.subtitle {
  margin-top: -0.5rem;
  margin-bottom: 1rem;
  color: var(--muted);
}

.card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.85rem;
  padding: 1rem 1.2rem;
  box-shadow: 0 18px 35px rgba(0, 0, 0, 0.2);
}

.card + .card {
  margin-top: 0.8rem;
}

.card-list {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 0.8rem;
}

.stat-card .label {
  color: var(--muted);
  font-size: 0.8rem;
}

.stat-card .value {
  font-size: 1.25rem;
  font-weight: 600;
}
</style>
