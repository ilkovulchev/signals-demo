<script setup>
import { Signal } from 'signal-polyfill'
import { ref, computed } from 'vue'

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

const count = new Signal.State(0)
const countVueRef = ref(0)

const double = new Signal.Computed(() => {
  console.log('🔄 Computing double...')
  return count.get() * 2
})

const doubleVueComp = computed(() => {
  return countVueRef.value * 2
})

function accessDouble() {
  console.log('👉 Accessing double:', double.get())
}

function increment() {
  countVueRef.value = count.get() + 1
  count.set(count.get() + 1)
  console.log('➕ Count incremented:', count.get())
  accessDouble()
}
</script>

<template>
  <div class="form-card">
    <div class="output">Count(Vue): {{ countVueRef }}</div>
    <div class="output">Double(Vue): {{ doubleVueComp }}</div>

    <button @click="increment">Increment Count</button>
    <button @click="accessDouble" class="btn-secondary">Access Double</button>
  </div>
</template>
