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
    <Transition name="cinematic" mode="out-in">
      <section
        v-if="remaining.totalSeconds > 0"
        key="countdown"
        class="memory-panel"
        aria-labelledby="page-title"
      >
        <div class="intro">
          <p class="eyebrow">D-DAY</p>
          <h1 id="page-title">Every hour brings me closer.</h1>
          <p class="lead">
            13 Nisan'dan beri geçen her saat, bizi yeniden birbirimize biraz daha
            yaklaştırıyor.
          </p>
        </div>

        <div class="timer-grid" aria-label="Deniz'e kavuşmaya kalan süre">
          <article v-for="part in timeParts" :key="part.label" class="time-card">
            <span class="time-value">{{ String(part.value).padStart(2, '0') }}</span>
            <span class="time-label">{{ part.label }}</span>
          </article>
        </div>

        <div class="reunion">
          <p class="date-line">06 July 2026 • 14:00</p>
          <p class="place-line">İzmir Adnan Menderes Airport</p>
        </div>

        <div class="progress-wrap" aria-label="Today to reunion progress">
          <div class="progress-meta">
            <span>Today</span>
            <span>Reunion</span>
          </div>
          <div class="progress-track">
            <div class="progress-fill" :style="{ width: `${progress}%` }"></div>
          </div>
        </div>

        <p class="closing-line">When this countdown ends, our forever begins.</p>
      </section>

      <section v-else key="completed" class="memory-panel completed-state" aria-live="polite">
        <div class="completed-glow" aria-hidden="true"></div>
        <div class="completed-copy">
          <p class="eyebrow">D-DAY</p>
          <h1>Welcome home, Deniz.</h1>
          <p>No more distance. Our forever starts now.</p>
        </div>
      </section>
    </Transition>
  </main>
</template>
