<template>
  <section>
    <h1>Range Sessions</h1>
    <p class="subtitle">
      One card per visit. Inside each session: stats for every club you used.
    </p>

    <!-- WIZARD CARD -->
    <div class="card wizard-card">
      <h2>Log Session</h2>
      <p class="wizard-step-label">
        Step {{ stepNumber }} of 4 · {{ stepTitle }}
      </p>

      <!-- STEP 1: DATE -->
      <div v-if="step === 'date'" class="wizard-step">
        <label class="field">
          Date
          <input v-model="wizard.date" type="date" />
        </label>

        <div class="wizard-actions">
          <button class="secondary-btn" disabled>Back</button>
          <button
            class="primary-btn"
            :disabled="!wizard.date"
            @click="goToClubs"
          >
            Next
          </button>
        </div>
      </div>

      <!-- STEP 2: CLUBS -->
      <div v-else-if="step === 'clubs'" class="wizard-step">
        <p class="muted">Which clubs did you actually hit this session?</p>
        <div class="club-checkboxes">
          <label
            v-for="club in clubs"
            :key="club.id"
            class="checkbox-pill"
          >
            <input
              type="checkbox"
              :value="club.id"
              v-model="wizard.selectedClubIds"
            />
            <span>{{ club.name }}</span>
          </label>
        </div>

        <div class="wizard-actions">
          <button class="secondary-btn" @click="step = 'date'">Back</button>
          <button
            class="primary-btn"
            :disabled="wizard.selectedClubIds.length === 0"
            @click="startClubFlow"
          >
            Next
          </button>
        </div>
      </div>

      <!-- STEP 3: PER-CLUB FLOW (balls -> avg -> best for each club) -->
      <div v-else-if="step === 'club'" class="wizard-step">
        <p class="muted">
          Club {{ currentClubIndex + 1 }} of {{ wizard.entries.length }}
        </p>
        <h3>{{ currentClubName }}</h3>

        <label class="field">
          {{ currentQuestion }}
          <input
            type="number"
            min="1"
            :placeholder="currentPlaceholder"
            :value="currentValue"
            @input="updateCurrentValue($event.target.value)"
          />
        </label>

        <div class="wizard-actions">
          <button class="secondary-btn" @click="prevClubStep">
            Back
          </button>
          <button
            class="primary-btn"
            :disabled="!currentValue"
            @click="nextClubStep"
          >
            Next
          </button>
        </div>
      </div>

      <!-- STEP 4: REVIEW -->
      <div v-else-if="step === 'review'" class="wizard-step">
        <p class="muted">Quick review before saving.</p>

        <table class="summary-table">
          <thead>
            <tr>
              <th>Club</th>
              <th>Balls</th>
              <th>Avg (yd)</th>
              <th>Best (yd)</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="entry in wizard.entries" :key="entry.clubId">
              <td>{{ clubName(entry.clubId) }}</td>
              <td>{{ entry.ballsHit }}</td>
              <td>{{ entry.averageDistance }}</td>
              <td>{{ entry.bestDistance }}</td>
            </tr>
          </tbody>
        </table>

        <label class="field" style="margin-top: 0.75rem;">
          Notes (optional)
          <textarea
            v-model="wizard.notes"
            rows="2"
            placeholder="Evening grind, focused on driver tempo"
          />
        </label>

        <div class="wizard-actions">
          <button class="secondary-btn" @click="backFromReview">
            Back
          </button>
          <button class="primary-btn" @click="saveSession">
            Save session
          </button>
        </div>
      </div>
    </div>

    <!-- SESSION LIST -->
    <div class="card-list">
      <article
        v-for="session in sessionsSorted"
        :key="session.id"
        class="card session-card"
      >
        <header class="session-header" @click="toggleExpanded(session.id)">
          <div>
            <h3>{{ formatDate(session.date) }}</h3>
            <p class="muted">
              {{ session.entries.length }} club<span v-if="session.entries.length > 1">s</span>
              · {{ totalBalls(session) }} balls
            </p>
          </div>
          <div class="session-right">
            <span class="pill small">
              Best: {{ bestShot(session) }} yd
            </span>
            <span class="clubs-inline">
              <span
                v-for="entry in session.entries"
                :key="entry.id"
                class="club-chip"
              >
                {{ clubName(entry.clubId) }}
              </span>
            </span>
          </div>
        </header>

        <div v-if="expandedSessionId === session.id" class="session-details">
          <table class="summary-table">
            <thead>
              <tr>
                <th>Club</th>
                <th>Balls</th>
                <th>Avg (yd)</th>
                <th>Best (yd)</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="entry in session.entries" :key="entry.id">
                <td>{{ clubName(entry.clubId) }}</td>
                <td>{{ entry.ballsHit }}</td>
                <td>{{ entry.averageDistance }}</td>
                <td>{{ entry.bestDistance }}</td>
              </tr>
            </tbody>
          </table>
          <p v-if="session.notes" class="notes">
            {{ session.notes }}
          </p>
        </div>
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
      step: 'date', // 'date' | 'clubs' | 'club' | 'review'
      wizard: {
        date: today,
        selectedClubIds: [],
        entries: [],
        notes: ''
      },
      currentClubIndex: 0,
      currentField: 'balls', // 'balls' | 'avg' | 'best'
      expandedSessionId: null
    }
  },

  computed: {
    sessionsSorted() {
      return [...this.sessions].sort((a, b) => b.date.localeCompare(a.date))
    },

    currentEntry() {
      return this.wizard.entries[this.currentClubIndex] || null
    },

    currentClubId() {
      return this.currentEntry ? this.currentEntry.clubId : null
    },

    currentClubName() {
      if (!this.currentClubId) return ''
      const club = this.clubs.find(c => c.id === this.currentClubId)
      return club ? club.name : 'Unknown club'
    },

    currentValue() {
      if (!this.currentEntry) return ''
      if (this.currentField === 'balls') return this.currentEntry.ballsHit
      if (this.currentField === 'avg') return this.currentEntry.averageDistance
      if (this.currentField === 'best') return this.currentEntry.bestDistance
      return ''
    },

    currentQuestion() {
      if (this.currentField === 'balls') {
        return 'How many balls did you hit with this club?'
      } else if (this.currentField === 'avg') {
        return 'What was your average distance with this club? (yd)'
      } else {
        return 'What was your best shot with this club? (yd)'
      }
    },

    currentPlaceholder() {
      if (this.currentField === 'balls') return 'e.g. 40'
      if (this.currentField === 'avg') return 'e.g. 235'
      if (this.currentField === 'best') return 'e.g. 270'
      return ''
    },

    stepNumber() {
      const map = { date: 1, clubs: 2, club: 3, review: 4 }
      return map[this.step] || 1
    },

    stepTitle() {
      switch (this.step) {
        case 'date':
          return 'Pick the date'
        case 'clubs':
          return 'Choose clubs you used'
        case 'club':
          return 'Fill stats for each club'
        case 'review':
          return 'Review & save'
        default:
          return ''
      }
    }
  },

  methods: {
    // --- WIZARD FLOW ---

    goToClubs() {
      if (!this.wizard.date) return
      this.step = 'clubs'
    },

    startClubFlow() {
      if (!this.wizard.selectedClubIds.length) return

      // Create per-club entries
      this.wizard.entries = this.wizard.selectedClubIds.map(id => ({
        clubId: id,
        ballsHit: null,
        averageDistance: null,
        bestDistance: null
      }))
      this.currentClubIndex = 0
      this.currentField = 'balls'
      this.step = 'club'
    },

    updateCurrentValue(raw) {
      if (!this.currentEntry) return
      const value = raw === '' ? null : Number(raw)
      if (Number.isNaN(value) || value <= 0) {
        if (this.currentField === 'balls') this.currentEntry.ballsHit = null
        if (this.currentField === 'avg') this.currentEntry.averageDistance = null
        if (this.currentField === 'best') this.currentEntry.bestDistance = null
        return
      }

      if (this.currentField === 'balls') this.currentEntry.ballsHit = value
      if (this.currentField === 'avg') this.currentEntry.averageDistance = value
      if (this.currentField === 'best') this.currentEntry.bestDistance = value
    },

    nextClubStep() {
      if (!this.currentValue) return

      // Move within club: balls -> avg -> best
      if (this.currentField === 'balls') {
        this.currentField = 'avg'
        return
      }
      if (this.currentField === 'avg') {
        this.currentField = 'best'
        return
      }

      // At 'best'
      if (this.currentField === 'best') {
        // If not last club: go to next club, back to balls
        if (this.currentClubIndex < this.wizard.entries.length - 1) {
          this.currentClubIndex++
          this.currentField = 'balls'
        } else {
          // Last club done -> review
          this.step = 'review'
        }
      }
    },

    prevClubStep() {
      // Go backwards within the flow
      if (this.currentField === 'balls') {
        // we're at first field for a club
        if (this.currentClubIndex === 0) {
          // jump back to club selection
          this.step = 'clubs'
        } else {
          // go to previous club, last field
          this.currentClubIndex--
          this.currentField = 'best'
        }
      } else if (this.currentField === 'avg') {
        this.currentField = 'balls'
      } else if (this.currentField === 'best') {
        this.currentField = 'avg'
      }
    },

    backFromReview() {
      // go back to last club / best field
      this.step = 'club'
      this.currentClubIndex = this.wizard.entries.length - 1
      this.currentField = 'best'
    },

    saveSession() {
      const payload = {
        date: this.wizard.date,
        notes: this.wizard.notes,
        entries: this.wizard.entries.map(e => ({ ...e }))
      }
      this.$emit('add-session', payload)
      this.resetWizard()
    },

    resetWizard() {
      const today = new Date().toISOString().slice(0, 10)
      this.step = 'date'
      this.wizard = {
        date: today,
        selectedClubIds: [],
        entries: [],
        notes: ''
      }
      this.currentClubIndex = 0
      this.currentField = 'balls'
    },

    // --- DISPLAY HELPERS ---

    clubName(id) {
      const club = this.clubs.find(c => c.id === id)
      return club ? club.name : 'Unknown club'
    },

    formatDate(date) {
      return new Date(date).toLocaleDateString()
    },

    totalBalls(session) {
      return (session.entries || []).reduce(
        (sum, e) => sum + (e.ballsHit || 0),
        0
      )
    },

    bestShot(session) {
      if (!session.entries || !session.entries.length) return 0
      return Math.max(...session.entries.map(e => e.bestDistance || 0))
    },

    toggleExpanded(id) {
      this.expandedSessionId = this.expandedSessionId === id ? null : id
    }
  }
}
</script>

<style scoped>
.wizard-card {
  margin-bottom: 1rem;
}

.wizard-step-label {
  font-size: 0.85rem;
  color: var(--muted);
  margin-bottom: 0.5rem;
}

.wizard-step h3 {
  margin-bottom: 0.4rem;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
  font-size: 0.9rem;
}

input,
textarea {
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 0.5rem;
  color: var(--text);
  padding: 0.5rem;
}

textarea {
  resize: vertical;
}

.wizard-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
  margin-top: 0.75rem;
}

.primary-btn {
  padding: 0.45rem 1.1rem;
  border-radius: 999px;
  border: none;
  background: var(--accent);
  color: #022c22;
  cursor: pointer;
  font-weight: 600;
}

.secondary-btn {
  padding: 0.45rem 1.1rem;
  border-radius: 999px;
  border: 1px solid var(--border);
  background: transparent;
  color: var(--text);
  cursor: pointer;
}

.primary-btn:disabled,
.secondary-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.club-checkboxes {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-top: 0.5rem;
}

.checkbox-pill {
  display: inline-flex;
  align-items: center;
  gap: 0.25rem;
  padding: 0.25rem 0.7rem;
  border-radius: 999px;
  border: 1px solid var(--border);
  cursor: pointer;
  font-size: 0.85rem;
}

.checkbox-pill input {
  accent-color: var(--accent);
}

/* sessions list */
.session-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
}

.session-right {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.25rem;
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

.clubs-inline {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
  justify-content: flex-end;
}

.club-chip {
  font-size: 0.75rem;
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
  border: 1px solid var(--border);
}

.session-details {
  margin-top: 0.6rem;
}

.summary-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.85rem;
}

.summary-table th,
.summary-table td {
  padding: 0.35rem 0.4rem;
  border-bottom: 1px solid var(--border);
  text-align: left;
}

.summary-table th {
  font-weight: 600;
}

.notes {
  margin-top: 0.5rem;
  font-size: 0.9rem;
}
</style>
