<template>
  <section>
    <h1>Your Stats</h1>
    <p class="subtitle">Across all sessions and clubs.</p>

    <div class="stats-grid">
      <div class="card stat-card">
        <p class="label">Total sessions</p>
        <p class="value">{{ totalSessions }}</p>
      </div>

      <div class="card stat-card">
        <p class="label">Total balls hit</p>
        <p class="value">{{ totalBalls }}</p>
      </div>

      <div class="card stat-card">
        <p class="label">Most used club</p>
        <p class="value">{{ mostUsedClubName || '—' }}</p>
      </div>

      <div class="card stat-card">
        <p class="label">Best shot</p>
        <p class="value">
          {{ bestShot ? bestShot + ' yd' : '—' }}
        </p>
      </div>
    </div>

    <div class="card" style="margin-top: 1rem;">
      <h2>Average distance per session</h2>
      <p class="subtitle">Graph coming soon.</p>
    </div>
  </section>
</template>

<script>
export default {
  props: {
    sessions: { type: Array, required: true },
    clubs: { type: Array, required: true }
  },

  computed: {
    totalSessions() {
      return this.sessions.length
    },

    allEntries() {
      return this.sessions.flatMap(s => s.entries || [])
    },

    totalBalls() {
      return this.allEntries.reduce((sum, e) => sum + (e.ballsHit || 0), 0)
    },

    bestShot() {
      if (!this.allEntries.length) return null
      return Math.max(...this.allEntries.map(e => e.bestDistance || 0))
    },

    mostUsedClubName() {
      if (!this.allEntries.length) return null

      const counts = {}
      for (const e of this.allEntries) {
        counts[e.clubId] = (counts[e.clubId] || 0) + (e.ballsHit || 0)
      }

      const entries = Object.entries(counts)
      if (!entries.length) return null

      const [topId] = entries.sort((a, b) => b[1] - a[1])[0]
      const club = this.clubs.find(c => c.id === Number(topId))
      return club ? club.name : null
    }
  }
}
</script>

<style scoped>
.label {
  color: var(--muted);
  font-size: 0.8rem;
}

.value {
  font-size: 1.25rem;
  font-weight: 600;
}
</style>
