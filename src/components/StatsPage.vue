<template>
  <section>
    <h1>Your Stats</h1>
    <p class="subtitle">Overall numbers across all logged entries.</p>

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

    <div class="card" style="margin-top: 1rem">
      <h2>Average distance per session</h2>
      <p class="subtitle">Graph coming soon.</p>
    </div>
  </section>
</template>

<script>
export default {
  props: {
    totalSessions: { type: Number, required: true },
    totalBalls: { type: Number, required: true },
    mostUsedClubName: { type: String, default: null },
    bestShot: { type: Number, default: null }
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
