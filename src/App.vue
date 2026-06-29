<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'

const startDate = new Date('2026-04-13T00:00:00+03:00')
const targetDate = new Date('2026-07-06T14:00:00+03:00')
const now = ref(new Date())
let intervalId

const remaining = computed(() => {
  const diff = Math.max(targetDate.getTime() - now.value.getTime(), 0)
  const totalSeconds = Math.floor(diff / 1000)

  return {
    totalSeconds,
    hours: Math.floor(totalSeconds / 3600),
    minutes: Math.floor((totalSeconds % 3600) / 60),
    seconds: totalSeconds % 60,
  }
})

const timeParts = computed(() => [
  { label: 'Total hours', value: remaining.value.hours },
  { label: 'Minutes', value: remaining.value.minutes },
  { label: 'Seconds', value: remaining.value.seconds },
])

const isComplete = computed(() => remaining.value.totalSeconds === 0)

const progress = computed(() => {
  const total = targetDate.getTime() - startDate.getTime()
  const elapsed = now.value.getTime() - startDate.getTime()

  return Math.min(Math.max((elapsed / total) * 100, 0), 100)
})

onMounted(() => {
  intervalId = window.setInterval(() => {
    now.value = new Date()
  }, 1000)
})

onUnmounted(() => {
  window.clearInterval(intervalId)
})
</script>

<template>
  <main class="page-shell">
    <section class="memory-panel" aria-labelledby="page-title">
      <div class="intro">
        <p class="eyebrow">D-DAY</p>
        <h1 id="page-title">We are choosing us again.</h1>
        <p class="lead">
          Every passing day has been quietly carrying us back to the love we chose again.
        </p>
      </div>

      <div class="timer-grid" aria-label="Time remaining until homecoming">
        <article v-for="part in timeParts" :key="part.label" class="time-card">
          <span class="time-value">{{ String(part.value).padStart(2, '0') }}</span>
          <span class="time-label">{{ part.label }}</span>
        </article>
      </div>

      <div class="reunion">
        <p class="date-line">06 July 2026 • 14:00</p>
        <p class="place-line">İzmir Adnan Menderes Airport</p>
      </div>

      <div class="progress-wrap" aria-label="April 13 to homecoming progress">
        <div class="progress-meta">
          <span>13 Apr 2026</span>
          <span>Homecoming</span>
        </div>
        <div class="progress-track">
          <div class="progress-fill" :style="{ width: `${progress}%` }"></div>
        </div>
        <p class="progress-note">
          13.04.26 to 06.07.26 — the day you come home.
        </p>
      </div>

      <p class="closing-line">When this countdown ends, our forever begins.</p>

      <Transition name="modal">
        <div v-if="isComplete" class="arrival-overlay" aria-live="polite">
          <div class="arrival-modal" role="dialog" aria-modal="true" aria-labelledby="arrival-title">
            <span class="heart" aria-hidden="true">❤️</span>
            <h2 id="arrival-title">Welcome Home</h2>
            <p>Deniz is here. No more distance.</p>
          </div>
        </div>
      </Transition>
    </section>
  </main>
</template>
