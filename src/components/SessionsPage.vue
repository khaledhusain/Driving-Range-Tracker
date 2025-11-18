<template>
  <section>
    <h1>Range Sessions</h1>
    <p class="subtitle">Log your work at the range (one club per entry).</p>

    <div class="card">
      <h2>Log Session</h2>
      <form @submit.prevent="submit" class="form-grid">
        <label>
          Date
          <input v-model="local.date" type="date" />
        </label>

        <label>
          Club
          <select v-model.number="local.clubId">
            <option
              v-for="club in clubs"
              :key="club.id"
              :value="club.id"
            >
              {{ club.name }}
            </option>
          </select>
        </label>

        <label>
          Balls hit
          <input v-model.number="local.ballsHit" type="number" placeholder="50" />
        </label>

        <label>
          Avg distance (yd)
          <input
            v-model.number="local.averageDistance"
            type="number"
            placeholder="235"
          />
        </label>

        <label>
          Best distance (yd)
          <input
            v-model.number="local.bestDistance"
            type="number"
            placeholder="260"
          />
        </label>

        <label class="full-width">
          Notes
          <textarea
            v-model="local.notes"
            rows="2"
            placeholder="Slices early, improved later"
          />
        </label>

        <button type="submit" class="primary-btn">Log session</button>
      </form>
    </div>

    <div class="card-list">
      <article
        v-for="session in sessionsSorted"
        :key="session.id"
        class="card"
      >
        <header class="session-header">
          <div>
            <h3>{{ formatDate(session.date) }}</h3>
            <p class="muted">{{ clubName(session.clubId) }}</p>
          </div>
          <span class="pill small">{{ session.ballsHit }} balls</span>
        </header>

        <p class="muted">
          Avg: <strong>{{ session.averageDistance }} yd</strong> ·
          Best: <strong>{{ session.bestDistance }} yd</strong>
        </p>
        <p v-if="session.notes" class="notes">{{ session.notes }}</p>
      </article>
    </div>
  </section>
</template>

<script>
export default {
  props: {
    clubs: { type: Array, required: true },
    sessions: { type: Array, required: true }
  },
  data() {
    const today = new Date().toISOString().slice(0, 10)
    return {
      local: {
        date: today,
        clubId: this.clubs[0]?.id || null,
        ballsHit: null,
        averageDistance: null,
        bestDistance: null,
        notes: ''
      }
    }
  },
  computed: {
    sessionsSorted() {
      return [...this.sessions].sort((a, b) => b.date.localeCompare(a.date))
    }
  },
  methods: {
    submit() {
      const { date, clubId, ballsHit, averageDistance, bestDistance, notes } =
        this.local

      if (!date || !clubId || !ballsHit || !averageDistance || !bestDistance) {
        return
      }

      this.$emit('add-session', {
        date,
        clubId,
        ballsHit,
        averageDistance,
        bestDistance,
        notes
      })

      this.local.ballsHit = null
      this.local.averageDistance = null
      this.local.bestDistance = null
      this.local.notes = ''
    },

    clubName(id) {
      const club = this.clubs.find(c => c.id === id)
      return club ? club.name : 'Unknown'
    },

    formatDate(date) {
      return new Date(date).toLocaleDateString()
    }
  }
}
</script>

<style scoped>
.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 0.8rem;
  margin-top: 0.75rem;
}

.full-width {
  grid-column: 1 / -1;
}

input,
textarea,
select {
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 0.5rem;
  color: var(--text);
  padding: 0.5rem;
}

/* 👇 Fix dark-mode dropdown options */
select {
  background-color: var(--card);
}

option {
  background-color: var(--card);
  color: var(--text);
}

textarea {
  resize: vertical;
}

.primary-btn {
  grid-column: 1 / -1;
  padding: 0.55rem 1.3rem;
  border-radius: 999px;
  border: none;
  background: var(--accent);
  color: #022c22;
  cursor: pointer;
}

.session-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.pill {
  padding: 0.2rem 0.7rem;
  border-radius: 999px;
  background: rgba(47, 122, 76, 0.2);
  color: var(--accent);
  font-size: 0.8rem;
}

.pill.small {
  font-size: 0.75rem;
}

.muted {
  color: var(--muted);
  font-size: 0.9rem;
}

.notes {
  margin-top: 0.3rem;
  font-size: 0.9rem;
}
</style>
