<template>
  <div class="draw">
    <h1>This canvas draw</h1>
    <canvas ref="canvas" id="draw" @click="canvasClick"></canvas>
  </div>
</template>

<script setup>
import { ref,  onMounted } from 'vue'

  // canvas draw and learn
  let canvas = ref(null)
  console.log(canvas)
  let ctx = {}
  onMounted(() => {
    ctx = canvas.value.getContext('2d')
    // play()
    ctx.beginPath()
    ctx.rect(20, 20, 150, 100)
    ctx.stroke()
    updateShape()
  })

  const updateShape = () => {
    ctx.beginPath()
    ctx.strokeStyle = 'red'
    ctx.rect(50, 20, 150, 100)
    ctx.stroke()
  }



</script>

<style scoped>
@media (min-width: 1024px) {
  #draw {
    border: 2px dashed green;
  }
}
</style>
