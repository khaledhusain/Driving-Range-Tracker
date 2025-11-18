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
        <div class="club-main">
          <div class="club-text">
            <h2 class="club-name">{{ club.name }}</h2>
            <p v-if="club.notes" class="club-notes">{{ club.notes }}</p>
          </div>
          <span class="distance-pill">
            {{ club.typicalDistance }} yd
          </span>
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
      }
    }
  },
  methods: {
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
    }
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

/* list container is centered, cards are full width inside it */
.bag-list {
  width: 100%;
  max-width: 800px;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

/* club card = full width brutalist rectangle */
.club-card {
  width: 100%;
  border: 1px solid var(--border);
  background: var(--card);
  padding: 1.1rem 1.3rem;
  border-radius: 2px;
  transition: transform 0.12s ease, border-color 0.12s ease,
    background-color 0.12s ease;
}

.club-card:hover {
  transform: translateY(-2px);
  border-color: var(--accent);
}

/* layout inside card */
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

.club-card:hover .distance-pill {
  background: rgba(47, 122, 76, 0.35);
}

/* floating add button */
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
</style>
