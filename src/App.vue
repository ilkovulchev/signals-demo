<script setup>
import { Signal } from 'signal-polyfill'

// --- Helper effect implementation ---
let needsEnqueue = true
const w = new Signal.subtle.Watcher(() => {
  if (needsEnqueue) {
    needsEnqueue = false
    queueMicrotask(processPending)
  }
})

function processPending() {
  needsEnqueue = true
  for (const s of w.getPending()) {
    s.get()
  }
  w.watch()
}

function effect(callback) {
  let cleanup
  const computed = new Signal.Computed(() => {
    if (typeof cleanup === 'function') cleanup()
    cleanup = callback()
  })
  w.watch(computed)
  computed.get()
  return () => {
    w.unwatch(computed)
    if (typeof cleanup === 'function') cleanup()
    cleanup = undefined
  }
}

// --- Demo code using the effect helper ---
const count = new Signal.State(0)
const double = new Signal.Computed(() => count.get() * 2)

effect(() => {
  console.log('Double is:', double.get())
})

// Update signals
count.set(1) // Logs: "Double is: 2"
count.set(2) // Logs: "Double is: 4"
</script>

<template>
  <div>Hello</div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
