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

// 1. Base signals
const firstName = new Signal.State('John')
const lastName = new Signal.State('Doe')

const middleName = new Signal.State('Bon')

const fullNameRef = ref('John Doe')
const firstNameRef = ref('John')
const lastNameRef = ref('Doe')
const middleNameRef = ref('Bon')

// 2. Computed signal (auto-tracks deps)
const fullName = new Signal.Computed(() => {
  console.log('Triggered computed signal')
  fullNameRef.value = `${firstName.get()} ${lastName.get()}`
  return `${firstName.get()} ${lastName.get()}`
})

// watch fullName using the custom effect
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
function changeMiddleName(e, value) {
  if (e.type === 'click' || (e.type === 'keypress' && e.key === 'Enter')) {
    middleName.set(value)
  }
}
function clearNames() {
  firstName.set('John')
  lastName.set('Doe')
  middleName.set('Bon')

  firstNameRef.value = 'John'
  lastNameRef.value = 'Doe'
  middleNameRef.value = 'Bon'
}
</script>

<template>
  <div class="form-card">
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

    <br />
    <label for="middleName">Middle Name: </label
    ><input
      id="middleName"
      v-model="middleNameRef"
      @keypress="changeMiddleName($event, middleNameRef)"
    />
    <button @click="changeMiddleName($event, middleNameRef)">Change Middle Name</button>

    <div class="output">Full Name: {{ fullNameRef }}</div>

    <button @click="clearNames">Clear Names</button>
  </div>
</template>
