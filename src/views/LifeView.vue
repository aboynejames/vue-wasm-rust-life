<template>
  <!--<h1>This is about life</h1>-->
  <button id="play-pause" @click="startGamelife">{{ controlGame }}</button>
  <div id="fps">{{ gameStats }}</div>
  <canvas :width=canWidth  :height=canHeight ref="canvas" id="game-of-life-canvas" @click="canvasClick($event)"></canvas>
</template>

<script setup>
import { ref,  onMounted, onUpdated } from 'vue'
import { Universe, Cell } from './../pkg2/wasm_game_of_life.js'
import { memory } from './../pkg2/wasm_game_of_life_bg.wasm'

  // game of life code
  let gameStats = ref('')
  let controlGame = ref('start')
  let canvas = ref('')
  const canWidth = ref(769)
  const canHeight = ref(769)
  let ctx = {}
  onMounted(() => {
    ctx = canvas.value.getContext('2d')
    // play()
  })

    const CELL_SIZE = 5 // px
    const GRID_COLOR = "#CCCCCC"
    const DEAD_COLOR = "#FFFFFF"
    const ALIVE_COLOR = "#000000"

    // Construct the universe, and get its width and height.
    const universe = Universe.new()
    const width = universe.width()
    const height = universe.height()

    // Give the canvas room for all of our cells and a 1px border
    // around each of them.
    canvas.height = (CELL_SIZE + 1) * height + 1
    canvas.width = (CELL_SIZE + 1) * width + 1

   // controls and game info
   const fps = new class {
    constructor() {
      this.stats = gameStats
      this.frames = []
      this.lastFrameTimeStamp = performance.now()
    }

    render() {
      // Convert the delta time since the last frame render into a measure
      // of frames per second.
      const now = performance.now()
      const delta = now - this.lastFrameTimeStamp
      this.lastFrameTimeStamp = now
      const fps = 1 / delta * 1000

      // Save only the latest 100 timings.
      this.frames.push(fps)
      if (this.frames.length > 100) {
        this.frames.shift()
      }

      // Find the max, min, and mean of our 100 latest timings.
      let min = Infinity
      let max = -Infinity
      let sum = 0
      for (let i = 0; i < this.frames.length; i++) {
        sum += this.frames[i]
        min = Math.min(this.frames[i], min)
        max = Math.max(this.frames[i], max)
      }
      let mean = sum / this.frames.length

      // Render the statistics.
      this.stats.value = `
        Frames per Second:
        latest = ${Math.round(fps)}
        avg of last 100 = ${Math.round(mean)}
        min of last 100 = ${Math.round(min)}
        max of last 100 = ${Math.round(max)}
      `.trim()
    }
  }

  let animationId = null;

  const renderLoop = () => {
    fps.render()

     drawGrid()
     drawCells()

    for (let i = 0; i < 9; i++) {
      universe.tick()
    }
    // if (animationId < 1000) {
       animationId = requestAnimationFrame(renderLoop)
    // }
  }

  const isPaused = () => {
    return animationId === null
  }


  const play = () => {
    controlGame.value = '⏸'
    renderLoop()
  }

  const pause = () => {
    controlGame.value = 'play'
    cancelAnimationFrame(animationId)
    animationId = null
  }

  const startGamelife = () => {
    if (isPaused()) {
      play()
    } else {
      pause()
    }
  }

  // draw the graphics on the canvas
  const drawGrid = () => {
    ctx.beginPath()
    ctx.strokeStyle = GRID_COLOR
    // Vertical lines.
    for (let i = 0; i <= width; i++) {
      ctx.moveTo(i * (CELL_SIZE + 1) + 1, 0)
      ctx.lineTo(i * (CELL_SIZE + 1) + 1, (CELL_SIZE + 1) * height + 1)
    }

    // Horizontal lines.
    for (let j = 0; j <= height; j++) {
      ctx.strokeStyle = GRID_COLOR
      ctx.moveTo(0,                           j * (CELL_SIZE + 1) + 1)
      ctx.lineTo((CELL_SIZE + 1) * width + 1, j * (CELL_SIZE + 1) + 1)
    }

    ctx.stroke()
  }

  const getIndex = (row, column) => {
    return row * width + column
  }

  const drawCells = () => {
    const cellsPtr = universe.cells()
      let cells = new Uint8Array(memory.buffer, cellsPtr, (height * width))

    ctx.beginPath()

    // Alive cells.
    ctx.fillStyle = ALIVE_COLOR
    for (let row = 0; row < height; row++) {
      for (let col = 0; col < width; col++) {
        const idx = getIndex(row, col)
        if (cells[idx] !== Cell.Alive) {
          continue
        }

        ctx.fillRect(
          col * (CELL_SIZE + 1) + 1,
          row * (CELL_SIZE + 1) + 1,
          CELL_SIZE,
          CELL_SIZE
        );
      }
    }

    // Dead cells.
    ctx.fillStyle = DEAD_COLOR
    for (let row = 0; row < height; row++) {
      for (let col = 0; col < width; col++) {
        const idx = getIndex(row, col)
        if (cells[idx] !== Cell.Dead) {
          continue
          // break
        }

        ctx.fillRect(
          col * (CELL_SIZE + 1) + 1,
          row * (CELL_SIZE + 1) + 1,
          CELL_SIZE,
          CELL_SIZE
        )
      }
    }

    ctx.stroke()
  }

    // canvas.addEventListener("click", event => {
    const canvasClick = function (event) {
      const boundingRect = canvas.value.getBoundingClientRect()
      const scaleX = canvas.width / boundingRect.width
      const scaleY = canvas.height / boundingRect.height

      const canvasLeft = (event.clientX - boundingRect.left) * scaleX
      const canvasTop = (event.clientY - boundingRect.top) * scaleY

      const row = Math.min(Math.floor(canvasTop / (CELL_SIZE + 1)), height - 1)
      const col = Math.min(Math.floor(canvasLeft / (CELL_SIZE + 1)), width - 1)

      universe.toggle_cell(row, col)

      drawCells()
      drawGrid()
    }

    // play()

</script>

<style>
  #play-pause {
    display: block;
  }

  #fps {
    display: block;
    font-family: monospace;
  }

  body {
  position: relative;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  /* min-height: 100vh;
  color: var(--color-text);
  background: var(--color-background);
  transition: color 0.5s, background-color 0.5s;
  line-height: 1.6;
  font-family: Inter, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
    Cantarell, 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif;
  font-size: 15px;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale; */
}

canvas#game-of-life-canvas2 {
  width: 800px;
  height: 800px;
}

@media (min-width: 1024px) {
}
</style>