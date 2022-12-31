<template>
  <div class="about">
    <h1>This is an about page</h1>
    <h2>Counter number = {{  counter.count  }}</h2>
    <button @click="pressCount">press me</button>
  </div>
</template>

<script setup>
// import { ref,  onMounted } from 'vue'
import init, { greet, add_heading } from './../pkg/learnwasm.js'
import { useCounterStore } from '@/stores/counter'

  init().then(() => {
   greet()
  })

  // counter
  const counter = useCounterStore()

  counter.count++
  // with autocompletion ✨
  counter.$patch({ count: counter.count + 1 })

  const pressCount = function () {
  // or using an action instead
    counter.increment()
  }


</script>

<style scoped>
 .about {
  display: grid;
 }
 
@media (min-width: 1024px) {
 .about {
  display: grid;
 }
}
</style>
