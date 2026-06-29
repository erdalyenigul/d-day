<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'

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
  { label: 'Saat', value: remaining.value.hours },
  { label: 'Dakika', value: remaining.value.minutes },
  { label: 'Saniye', value: remaining.value.seconds },
])

const progress = computed(() => {
  const startDate = new Date('2026-06-29T00:00:00+03:00')
  const total = targetDate.getTime() - startDate.getTime()
  const elapsed = now.value.getTime() - startDate.getTime()

  return Math.min(Math.max((elapsed / total) * 100, 0), 100)
})

const targetLabel = new Intl.DateTimeFormat('tr-TR', {
  day: 'numeric',
  month: 'long',
  year: 'numeric',
  hour: '2-digit',
  minute: '2-digit',
  timeZone: 'Europe/Istanbul',
}).format(targetDate)

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
    <section class="countdown-panel" aria-labelledby="page-title">
      <div class="status-row">
        <span class="status-dot" aria-hidden="true"></span>
        <span>Canlı geri sayım</span>
      </div>

      <div class="headline">
        <p class="eyebrow">D-Day</p>
        <h1 id="page-title">6 Temmuz 14:00</h1>
        <p class="target-date">{{ targetLabel }} · Türkiye saati</p>
      </div>

      <div v-if="remaining.totalSeconds > 0" class="timer-grid">
        <article v-for="part in timeParts" :key="part.label" class="time-card">
          <span class="time-value">{{ String(part.value).padStart(2, '0') }}</span>
          <span class="time-label">{{ part.label }}</span>
        </article>
      </div>

      <div v-else class="completed-state">
        <span class="completed-value">00:00:00</span>
        <span class="completed-label">Zaman geldi.</span>
      </div>

      <div class="progress-wrap" aria-label="Geri sayım ilerlemesi">
        <div class="progress-meta">
          <span>Bugunden hedefe</span>
          <span>{{ Math.round(progress) }}%</span>
        </div>
        <div class="progress-track">
          <div class="progress-fill" :style="{ width: `${progress}%` }"></div>
        </div>
      </div>
    </section>
  </main>
</template>
