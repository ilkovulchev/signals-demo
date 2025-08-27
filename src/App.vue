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

// --- Demo code for AUTOMATIC DEPNDENCY TRACKING ---
/*
// 1. Base signals
const firstName = new Signal.State('John')
const lastName = new Signal.State('Doe')

const fullNameRef = ref('John Doe')
const firstNameRef = ref('John')
const lastNameRef = ref('Doe')

// 2. Computed signal (auto-tracks deps)
const fullName = new Signal.Computed(() => {
  console.log('Triggered computed signal')
  fullNameRef.value = `${firstName.get()} ${lastName.get()}`
  return `${firstName.get()} ${lastName.get()}`
})

// watch fullName using your custom effect
effect(() => {
  console.log('Effect triggered:', fullName.get())
})

// 3. Event handlers
function changeFirstName(e, value) {
  if (e.type === 'click' || (e.type === 'keypress' && e.key === 'Enter')) {
    firstName.set(value)
  }
}
function changeLastName(e, value) {
  if (e.type === 'click' || (e.type === 'keypress' && e.key === 'Enter')) {
    lastName.set(value)
  }
}
function clearNames() {
  firstName.set('John')
  lastName.set('Doe')
}
*/

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

// --- Demo code for LAZY EVALUATION ---
/*
const count = new Signal.State(0)
const double = new Signal.Computed(() => count.get() * 2)

const countRef = ref(0)
const doubleRef = ref(0)
const updateCount = () => {
  count.set(count.get() + 1)

  countRef.value = count.get()
  doubleRef.value = double.get()

  console.log('Double is:', double.get())
}
*/

// --- Demo code for MEMOIZATION ---
/*
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
}
*/
</script>

<template>
  <!-- Demo code for AUTOMATIC DEPNDENCY TRACKING -->
  <!-- <div class="form-card">
    <label for="firstName">First Name: </label
    ><input
      id="firstName"
      v-model="firstNameRef"
      @keypress="changeFirstName($event, firstNameRef)"
    />
    <button @click="changeFirstName($event, firstNameRef)">Change First Name</button>
    <br />
    <label for="lastName">Lat Name: </label
    ><input id="lastName" v-model="lastNameRef" @keypress="changeLastName($event, lastNameRef)" />
    <button @click="changeLastName($event, lastNameRef)">Change Last Name</button>

    <div class="output">Full Name: {{ fullNameRef }}</div>

    <button @click="clearNames">Clear Names</button>
  </div> -->

  <!-- Demo code for LAZY EVALUATION -->
  <!-- <div class="form-card">
    <button @click="updateCount">Increase Count</button>

    <div class="output">Count: {{ countRef }}</div>
    <div class="output">Double: {{ doubleRef }}</div>
  </div> -->

  <!-- Demo code for MEMOIZATION -->
  <!-- <div class="form-card">
    <div class="output">Count(Vue): {{ countVueRef }}</div>
    <div class="output">Double(Vue): {{ doubleVueComp }}</div>

    <button @click="increment">Increment Count</button>
    <button @click="accessDouble" class="btn-secondary">Access Double</button>
  </div> -->
</template>

<style scoped></style>
