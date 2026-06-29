<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'

const targetDate = new Date('2026-07-06T14:00:00+03:00')
const progressWindowMs = (163 * 60 + 52) * 60 * 1000
const startDate = new Date(targetDate.getTime() - progressWindowMs)
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
  { label: 'TOTAL HOURS', value: remaining.value.hours },
  { label: 'MINUTES', value: remaining.value.minutes },
  { label: 'SECONDS', value: remaining.value.seconds },
])

const isComplete = computed(() => remaining.value.totalSeconds === 0)

const progress = computed(() => {
  const total = targetDate.getTime() - startDate.getTime()
  const elapsed = now.value.getTime() - startDate.getTime()

  return Math.min(Math.max((elapsed / total) * 100, 0), 100)
})

const progressPercent = computed(() => `${progress.value.toFixed(1)}%`)

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
          the day you come home.
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
        <p class="place-line">Airport Meet</p>
      </div>

      <div class="progress-wrap" aria-label="Countdown window progress">
        <div class="progress-meta">
          <span>AWAY</span>
          <span>{{ progressPercent }}</span>
          <span>HOME</span>
        </div>
        <div class="progress-track">
          <div class="progress-fill" :style="{ width: `${progress}%` }"></div>
        </div>
        <p class="progress-note">
          the day you come home.
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
