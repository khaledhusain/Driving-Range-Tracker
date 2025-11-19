<template>
  <section class="bag-page">
    <!-- Header -->
    <h1 class="bag-title">YOUR BAG</h1>

    <!-- Club list -->
    <div class="bag-list">
      <article
        v-for="club in clubs"
        :key="club.id"
        class="club-card"
      >
        <!-- Red delete background, fully inside card -->
        <div
          class="delete-bg"
          :class="{ open: swipeState[club.id]?.open }"
          @click="handleDelete(club.id)"
        >
          <span class="trash-icon">🗑</span>
        </div>

        <!-- Swipeable content -->
        <div
          class="swipe-track"
          :class="{ dragging: swipeState[club.id]?.dragging }"
          :style="{
            transform: `translateX(${swipeState[club.id]?.offset || 0}px)`
          }"
          @touchstart="onTouchStart(club.id, $event)"
          @touchmove="onTouchMove(club.id, $event)"
          @touchend="onTouchEnd(club.id)"
          @mousedown="onMouseDown(club.id, $event)"
        >
          <div class="club-main">
            <div class="club-text">
              <h2 class="club-name">{{ club.name }}</h2>
              <p v-if="club.notes" class="club-notes">{{ club.notes }}</p>
            </div>
            <span class="distance-pill">
              {{ club.typicalDistance }} yd
            </span>
          </div>
        </div>
      </article>
    </div>

    <!-- Floating add button -->
    <button class="fab" @click="openModal">+</button>

    <!-- Modal -->
    <div
      v-if="showModal"
      class="modal-backdrop"
      @click.self="closeModal"
    >
      <div class="modal-card">
        <header class="modal-header">
          <h2>Add Club</h2>
          <button class="modal-close" @click="closeModal">
            ×
          </button>
        </header>

        <form class="modal-form" @submit.prevent="submit">
          <label class="field">
            Name
            <input
              v-model="form.name"
              type="text"
              placeholder="7 Iron"
            />
          </label>

          <label class="field">
            Typical distance (yd)
            <input
              v-model.number="form.distance"
              type="number"
              min="1"
              placeholder="165"
            />
          </label>

          <label class="field">
            Notes
            <textarea
              v-model="form.notes"
              rows="2"
              placeholder="Reliable from 150–170"
            />
          </label>

          <button type="submit" class="primary-btn">
            Add club
          </button>
        </form>
      </div>
    </div>
  </section>
</template>

<script>
const MAX_REVEAL = 140      // how far left you can swipe (more than before)
const OPEN_THRESHOLD = -70  // how much you need to swipe to "lock" it open

export default {
  props: {
    clubs: { type: Array, required: true }
  },
  data() {
    return {
      showModal: false,
      form: {
        name: '',
        distance: null,
        notes: ''
      },
      swipeState: {},       // per-club swipe info
      mouseDraggingId: null // for desktop drag
    }
  },
  methods: {
    /* -------- Add club modal -------- */
    openModal() {
      this.showModal = true
    },
    closeModal() {
      this.showModal = false
      this.form = { name: '', distance: null, notes: '' }
    },
    submit() {
      if (!this.form.name || !this.form.distance) return

      this.$emit('add-club', {
        name: this.form.name.trim(),
        typicalDistance: this.form.distance,
        notes: this.form.notes.trim()
      })
      this.closeModal()
    },

    handleDelete(id) {
      this.$emit('delete-club', id)
      const copy = { ...this.swipeState }
      delete copy[id]
      this.swipeState = copy
    },

    ensureSwipeState(id) {
      if (!this.swipeState[id]) {
        this.swipeState = {
          ...this.swipeState,
          [id]: {
            startX: 0,
            offset: 0,
            dragging: false,
            open: false,
            resetTimer: null
          }
        }
      }
      return this.swipeState[id]
    },

    scheduleReset(id) {
      const state = this.ensureSwipeState(id)

      if (state.resetTimer) {
        clearTimeout(state.resetTimer)
      }

      state.resetTimer = setTimeout(() => {
        const s = this.ensureSwipeState(id)
        if (!s.open) return
        s.offset = 0        // slide back
        s.open = false
        s.resetTimer = null
      }, 2200)              // a bit slower
    },

    /* -------- Touch swipe handlers -------- */
    onTouchStart(id, event) {
      const state = this.ensureSwipeState(id)
      state.startX = event.touches[0].clientX
      state.dragging = true
    },
    onTouchMove(id, event) {
      const state = this.ensureSwipeState(id)
      if (!state.dragging) return

      const currentX = event.touches[0].clientX
      const deltaX = currentX - state.startX

      if (deltaX < 0) {
        state.offset = Math.max(deltaX, -MAX_REVEAL)
      } else {
        state.offset = Math.min(deltaX, 0)
      }
    },
    onTouchEnd(id) {
      const state = this.ensureSwipeState(id)

      if (state.offset <= OPEN_THRESHOLD) {
        state.offset = -MAX_REVEAL
        state.open = true
        this.scheduleReset(id)
      } else {
        state.offset = 0
        state.open = false
      }
      state.dragging = false
    },

    /* -------- Mouse swipe handlers (desktop) -------- */
    onMouseDown(id, event) {
      const state = this.ensureSwipeState(id)
      state.startX = event.clientX
      state.dragging = true
      this.mouseDraggingId = id
      window.addEventListener('mousemove', this.onMouseMove)
      window.addEventListener('mouseup', this.onMouseUp)
    },
    onMouseMove(event) {
      const id = this.mouseDraggingId
      if (!id) return
      const state = this.ensureSwipeState(id)
      if (!state.dragging) return

      const currentX = event.clientX
      const deltaX = currentX - state.startX

      if (deltaX < 0) {
        state.offset = Math.max(deltaX, -MAX_REVEAL)
      } else {
        state.offset = Math.min(deltaX, 0)
      }
    },
    onMouseUp() {
      const id = this.mouseDraggingId
      if (!id) return
      const state = this.ensureSwipeState(id)

      if (state.offset <= OPEN_THRESHOLD) {
        state.offset = -MAX_REVEAL
        state.open = true
        this.scheduleReset(id)
      } else {
        state.offset = 0
        state.open = false
      }
      state.dragging = false
      this.mouseDraggingId = null
      window.removeEventListener('mousemove', this.onMouseMove)
      window.removeEventListener('mouseup', this.onMouseUp)
    }
  },
  beforeUnmount() {
    window.removeEventListener('mousemove', this.onMouseMove)
    window.removeEventListener('mouseup', this.onMouseUp)
    Object.values(this.swipeState).forEach(state => {
      if (state.resetTimer) clearTimeout(state.resetTimer)
    })
  }
}
</script>

<style scoped>
.bag-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.75rem;
}

.bag-title {
  text-align: center;
  font-size: 2.1rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

/* centered column of cards */
.bag-list {
  width: 100%;
  max-width: 800px;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

/* outer wrapper: border, background, no bleed */
.club-card {
  position: relative;
  width: 100%;
  border: 1px solid var(--border);
  background: var(--card);
  border-radius: 2px;
  overflow: hidden; /* prevents swipe bleed */
}

/* red delete panel INSIDE card */
.delete-bg {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  width: 140px;
  background: #ef4444;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 1.4rem;
}

/* trash animation when open */
.delete-bg.open .trash-icon {
  animation: trash-pop 0.3s ease-out;
}

@keyframes trash-pop {
  0% {
    transform: scale(0.7);
    opacity: 0;
  }
  60% {
    transform: scale(1.15);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

/* movable content layer */
.swipe-track {
  position: relative;
  z-index: 1;
  width: 100%;
  box-sizing: border-box;
  padding: 1.1rem 1.3rem;
  transition: transform 0.28s ease; /* slower slide back */
}

.swipe-track.dragging {
  transition: none;
}

.club-main {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.club-text {
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
}

.club-name {
  font-size: 1.05rem;
  font-weight: 600;
}

.club-notes {
  font-size: 0.9rem;
  color: var(--muted);
}

/* distance pill */
.distance-pill {
  padding: 0.25rem 0.8rem;
  border-radius: 999px;
  font-size: 0.85rem;
  background: rgba(47, 122, 76, 0.18);
  color: var(--accent);
  white-space: nowrap;
}

.swipe-track:hover .distance-pill {
  background: rgba(47, 122, 76, 0.35);
}

/* floating green + */
.fab {
  position: fixed;
  right: 2rem;
  bottom: 2rem;
  width: 54px;
  height: 54px;
  border-radius: 50%;
  border: none;
  background: var(--accent);
  color: #ffffff;
  font-size: 2rem;
  line-height: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.35);
  transition: transform 0.12s ease, box-shadow 0.12s ease,
    filter 0.12s ease;
}

.fab:hover {
  transform: translateY(-2px);
  box-shadow: 0 14px 30px rgba(0, 0, 0, 0.4);
  filter: brightness(1.05);
}

/* modal */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 40;
}

.modal-card {
  width: 100%;
  max-width: 420px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 1.1rem 1.3rem 1.2rem;
  box-shadow: 0 20px 45px rgba(0, 0, 0, 0.5);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.75rem;
}

.modal-header h2 {
  font-size: 1.1rem;
}

.modal-close {
  background: transparent;
  border: none;
  color: var(--text);
  font-size: 1.4rem;
  cursor: pointer;
  line-height: 1;
}

.modal-form {
  display: flex;
  flex-direction: column;
  gap: 0.7rem;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  font-size: 0.9rem;
}

input,
textarea {
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 4px;
  color: var(--text);
  padding: 0.45rem 0.55rem;
}

textarea {
  resize: vertical;
}

.primary-btn {
  align-self: flex-end;
  margin-top: 0.2rem;
  padding: 0.45rem 1.1rem;
  border-radius: 999px;
  border: none;
  background: var(--accent);
  color: #022c22;
  cursor: pointer;
  font-weight: 600;
  font-size: 0.9rem;
}
.club-card {
  background: var(--card);      /* solid */
}

.swipe-track {
  background: var(--card);      /* solid */
}

</style>
