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
        Step {{ stepNumber }} of 5 · {{ stepTitle }}
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
            @click="startBallsStep"
          >
            Next
          </button>
        </div>
      </div>

      <!-- STEP 3: BALLS PER CLUB -->
      <div v-else-if="step === 'balls'" class="wizard-step">
        <p class="muted">
          Club {{ currentIndex + 1 }} of {{ wizard.entries.length }}
        </p>
        <h3>{{ currentClubName }}</h3>

        <label class="field">
          How many balls did you hit with this club?
          <input
            v-model.number="wizard.entries[currentIndex].ballsHit"
            type="number"
            min="1"
            placeholder="e.g. 40"
          />
        </label>

        <div class="wizard-actions">
          <button class="secondary-btn" @click="prevClubOrBackFromBalls">
            Back
          </button>
          <button
            class="primary-btn"
            :disabled="!wizard.entries[currentIndex].ballsHit"
            @click="nextBallsStep"
          >
            Next
          </button>
        </div>
      </div>

      <!-- STEP 4: AVG DISTANCE PER CLUB -->
      <div v-else-if="step === 'avg'" class="wizard-step">
        <p class="muted">
          Club {{ currentIndex + 1 }} of {{ wizard.entries.length }}
        </p>
        <h3>{{ currentClubName }}</h3>

        <label class="field">
          What was your average distance with this club? (yd)
          <input
            v-model.number="wizard.entries[currentIndex].averageDistance"
            type="number"
            min="1"
            placeholder="e.g. 235"
          />
        </label>

        <div class="wizard-actions">
          <button class="secondary-btn" @click="prevClubOrBackFromAvg">
            Back
          </button>
          <button
            class="primary-btn"
            :disabled="!wizard.entries[currentIndex].averageDistance"
            @click="nextAvgStep"
          >
            Next
          </button>
        </div>
      </div>

      <!-- STEP 5: BEST DISTANCE PER CLUB -->
      <div v-else-if="step === 'best'" class="wizard-step">
        <p class="muted">
          Club {{ currentIndex + 1 }} of {{ wizard.entries.length }}
        </p>
        <h3>{{ currentClubName }}</h3>

        <label class="field">
          What was your best shot with this club? (yd)
          <input
            v-model.number="wizard.entries[currentIndex].bestDistance"
            type="number"
            min="1"
            placeholder="e.g. 270"
          />
        </label>

        <div class="wizard-actions">
          <button class="secondary-btn" @click="prevClubOrBackFromBest">
            Back
          </button>
          <button
            class="primary-btn"
            :disabled="!wizard.entries[currentIndex].bestDistance"
            @click="nextBestStep"
          >
            Next
          </button>
        </div>
      </div>

      <!-- STEP 6: REVIEW -->
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
          <button class="secondary-btn" @click="step = 'best'">
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
      step: 'date', // 'date' | 'clubs' | 'balls' | 'avg' | 'best' | 'review'
      wizard: {
        date: today,
        selectedClubIds: [],
        entries: [],
        notes: ''
      },
      currentIndex: 0,
      expandedSessionId: null
    }
  },

  computed: {
    sessionsSorted() {
      return [...this.sessions].sort((a, b) => b.date.localeCompare(a.date))
    },

    currentClubId() {
      const entry = this.wizard.entries[this.currentIndex]
      return entry ? entry.clubId : null
    },

    currentClubName() {
      if (!this.currentClubId) return ''
      const club = this.clubs.find(c => c.id === this.currentClubId)
      return club ? club.name : 'Unknown club'
    },

    stepNumber() {
      const map = { date: 1, clubs: 2, balls: 3, avg: 4, best: 5, review: 6 }
      return map[this.step] || 1
    },

    stepTitle() {
      switch (this.step) {
        case 'date':
          return 'Pick the date'
        case 'clubs':
          return 'Choose clubs you used'
        case 'balls':
          return 'Balls hit per club'
        case 'avg':
          return 'Average distance per club'
        case 'best':
          return 'Best shot per club'
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

    startBallsStep() {
      if (!this.wizard.selectedClubIds.length) return

      // create entries skeleton
      this.wizard.entries = this.wizard.selectedClubIds.map(id => ({
        clubId: id,
        ballsHit: null,
        averageDistance: null,
        bestDistance: null
      }))
      this.currentIndex = 0
      this.step = 'balls'
    },

    nextBallsStep() {
      if (!this.wizard.entries[this.currentIndex].ballsHit) return

      if (this.currentIndex < this.wizard.entries.length - 1) {
        this.currentIndex++
      } else {
        this.currentIndex = 0
        this.step = 'avg'
      }
    },

    prevClubOrBackFromBalls() {
      if (this.currentIndex > 0) {
        this.currentIndex--
      } else {
        this.step = 'clubs'
      }
    },

    nextAvgStep() {
      if (!this.wizard.entries[this.currentIndex].averageDistance) return

      if (this.currentIndex < this.wizard.entries.length - 1) {
        this.currentIndex++
      } else {
        this.currentIndex = 0
        this.step = 'best'
      }
    },

    prevClubOrBackFromAvg() {
      if (this.currentIndex > 0) {
        this.currentIndex--
      } else {
        this.step = 'balls'
        this.currentIndex = this.wizard.entries.length - 1
      }
    },

    prevClubOrBackFromBest() {
      if (this.currentIndex > 0) {
        this.currentIndex--
      } else {
        this.step = 'avg'
        this.currentIndex = this.wizard.entries.length - 1
      }
    },

    nextBestStep() {
  if (!this.wizard.entries[this.currentIndex].bestDistance) return

  if (this.currentIndex < this.wizard.entries.length - 1) {
    // go to next club in "best" step
    this.currentIndex++
  } else {
    // last club done → go to review screen
    this.step = 'review'
  }
}
,

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
      this.currentIndex = 0
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
textarea,
select {
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 0.5rem;
  color: var(--text);
  padding: 0.5rem;
}

/* dark-mode dropdown fix */
select,
option {
  background-color: var(--card);
  color: var(--text);
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
