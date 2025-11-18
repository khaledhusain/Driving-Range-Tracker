<template>
  <section>
    <h1>My Bag</h1>
    <p class="subtitle">These are the clubs you’re grinding with.</p>

    <div class="card">
      <h2>Add Club</h2>
      <form @submit.prevent="submit" class="form-grid">
        <label>
          Name
          <input v-model="localClub.name" type="text" placeholder="Driver" />
        </label>
        <label>
          Typical distance (yd)
          <input
            v-model.number="localClub.typicalDistance"
            type="number"
            placeholder="250"
          />
        </label>
        <label class="full-width">
          Notes
          <textarea
            v-model="localClub.notes"
            rows="2"
            placeholder="My go-to club"
          />
        </label>
        <button type="submit" class="primary-btn">Add club</button>
      </form>
    </div>

    <div class="card-list">
      <article v-for="club in clubs" :key="club.id" class="card">
        <header class="club-header">
          <h3>{{ club.name }}</h3>
          <span class="pill">{{ club.typicalDistance }} yd</span>
        </header>
        <p class="muted" v-if="club.notes">{{ club.notes }}</p>
      </article>
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
      localClub: {
        name: '',
        typicalDistance: null,
        notes: ''
      }
    }
  },
  methods: {
    submit() {
      if (!this.localClub.name || !this.localClub.typicalDistance) return
      this.$emit('add-club', { ...this.localClub })
      this.localClub = { name: '', typicalDistance: null, notes: '' }
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

.primary-btn {
  grid-column: 1 / -1;
  padding: 0.55rem 1.3rem;
  border-radius: 999px;
  border: none;
  background: var(--accent);
  color: #022c22;
  cursor: pointer;
}

.club-header {
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

.muted {
  color: var(--muted);
  font-size: 0.9rem;
}
</style>
