<script setup>
import { Signal } from 'signal-polyfill'
import { ref } from 'vue'

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

// --- Demo code for Signal's effect ---
/*
let effectCount = 0

effect(() => {
  effectCount++
  console.log('Double is:', double.get())
  console.log('the effect was called', effectCount, 'times')
  countRef.value = count.get()
  doubleRef.value = double.get()
})
*/

const count = new Signal.State(0)
const double = new Signal.Computed(() => {
  console.log('🔄 Computing double...')
  return count.get() * 2
})

const countRef = ref(0)
const doubleRef = ref(0)
const increaseCount = () => {
  count.set(count.get() + 1)

  countRef.value = count.get()
  doubleRef.value = double.get()

  console.log('Double is:', double.get())
}
</script>

<template>
  <div class="form-card">
    <button @click="increaseCount">Increase Count</button>

    <div class="output">Count: {{ countRef }}</div>
    <div class="output">Double: {{ doubleRef }}</div>
  </div>
</template>
