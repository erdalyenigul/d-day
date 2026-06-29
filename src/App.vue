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
  { label: 'HOURS', value: remaining.value.hours },
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

const bokehColors = [
  'rgba(255, 0, 48, 0.92)',
  'rgba(255, 0, 36, 0.9)',
  'rgba(255, 18, 72, 0.86)',
  'rgba(220, 0, 34, 0.82)',
  'rgba(255, 48, 32, 0.82)',
  'rgba(0, 128, 255, 0.78)',
  'rgba(0, 92, 255, 0.72)',
  'rgba(118, 58, 255, 0.68)',
  'rgba(178, 54, 255, 0.6)',
  'rgba(0, 255, 170, 0.54)',
  'rgba(255, 128, 0, 0.9)',
  'rgba(255, 154, 24, 0.84)',
  'rgba(255, 210, 0, 0.78)',
  'rgba(255, 188, 72, 0.7)',
]

const bokehSizes = [58, 82, 108, 138, 172]

const randomBetween = (min, max) => min + Math.random() * (max - min)
const randomItem = (items) => items[Math.floor(Math.random() * items.length)]

const createLight = (index, overrides = {}) => {
  const angle = randomBetween(0, Math.PI * 2)
  const distanceA = randomBetween(24, 86)
  const distanceB = randomBetween(22, 78)
  const scatter = randomBetween(72, 190)

  return {
    id: index,
    x: `${randomBetween(2, 98).toFixed(1)}%`,
    y: `${randomBetween(6, 94).toFixed(1)}%`,
    size: `${randomItem(bokehSizes)}px`,
    color: randomItem(bokehColors),
    delay: `${-randomBetween(0, 7.5).toFixed(2)}s`,
    duration: `${randomBetween(5, 8.4).toFixed(2)}s`,
    dxA: `${Math.round(Math.cos(angle) * distanceA)}px`,
    dyA: `${Math.round(Math.sin(angle) * distanceA)}px`,
    dxB: `${Math.round(Math.cos(angle + randomBetween(1.2, 2.8)) * distanceB)}px`,
    dyB: `${Math.round(Math.sin(angle + randomBetween(1.2, 2.8)) * distanceB)}px`,
    scatterX: `${Math.round(Math.cos(angle + randomBetween(0.4, 1.2)) * scatter)}px`,
    scatterY: `${Math.round(Math.sin(angle + randomBetween(0.4, 1.2)) * scatter)}px`,
    born: false,
    ...overrides,
  }
}

const bokehLights = ref(Array.from({ length: 40 }, (_, index) => createLight(index)))
const bokehBursts = ref([])

const spawnLight = (parent) => {
  const index = bokehLights.value.length
  const angle = Math.random() * Math.PI * 2
  const spread = 0.8 + Math.random() * 2.2
  const x = Math.min(Math.max(parseFloat(parent.x) + Math.cos(angle) * spread, 2), 98)
  const y = Math.min(Math.max(parseFloat(parent.y) + Math.sin(angle) * spread, 6), 94)
  const size = randomItem(bokehSizes)
  const driftAngle = Math.random() * Math.PI * 2
  const distanceA = 28 + Math.random() * 58
  const distanceB = 24 + Math.random() * 54
  const burstId = `burst-${Date.now()}-${Math.random()}`

  bokehBursts.value.push({
    id: burstId,
    x: parent.x,
    y: parent.y,
    size: parent.size,
    color: parent.color,
  })

  window.setTimeout(() => {
    bokehBursts.value = bokehBursts.value.filter((burst) => burst.id !== burstId)
  }, 680)

  bokehLights.value.push(createLight(index, {
    id: `${index}-${Date.now()}`,
    x: `${x.toFixed(1)}%`,
    y: `${y.toFixed(1)}%`,
    size: `${size}px`,
    color: randomItem(bokehColors),
    delay: '0s',
    duration: `${5 + Math.random() * 3.2}s`,
    dxA: `${Math.round(Math.cos(driftAngle) * distanceA)}px`,
    dyA: `${Math.round(Math.sin(driftAngle) * distanceA)}px`,
    dxB: `${Math.round(Math.cos(driftAngle + 2.1) * distanceB)}px`,
    dyB: `${Math.round(Math.sin(driftAngle + 2.1) * distanceB)}px`,
    born: true,
  }))
}

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
    <div class="bokeh-layer" aria-hidden="true">
      <span
        v-for="light in bokehLights"
        :key="light.id"
        class="bokeh-light"
        :class="{ 'is-born': light.born }"
        :style="{
          left: light.x,
          top: light.y,
          width: light.size,
          height: light.size,
          '--bokeh-color': light.color,
          '--bokeh-delay': light.delay,
          '--bokeh-duration': light.duration,
          '--dx-a': light.dxA,
          '--dy-a': light.dyA,
          '--dx-b': light.dxB,
          '--dy-b': light.dyB,
          '--scatter-x': light.scatterX,
          '--scatter-y': light.scatterY,
        }"
        @click.stop="spawnLight(light)"
      ></span>

      <span
        v-for="burst in bokehBursts"
        :key="burst.id"
        class="bokeh-burst"
        :style="{
          left: burst.x,
          top: burst.y,
          width: burst.size,
          height: burst.size,
          '--bokeh-color': burst.color,
        }"
      ></span>
    </div>

    <section class="memory-panel" aria-labelledby="page-title">
      <div class="intro">
        <p class="eyebrow">D-DAY</p>
        <h1 id="page-title">THE DAY YOU<br> COME HOME</h1>
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
          Tap a Color, Add More Love.
        </p>
      </div>

      <Transition name="modal">
        <div v-if="isComplete" class="arrival-overlay" aria-live="polite">
          <div class="arrival-modal" role="dialog" aria-modal="true" aria-labelledby="arrival-title">
            <span class="heart" aria-hidden="true">❤️</span>
            <h2 id="arrival-title">Welcome Home</h2>
            <p>My Baby is here. No more distance.</p>
          </div>
        </div>
      </Transition>
    </section>
  </main>
</template>
