<template>
    <div class="canvas-container">
      <canvas id="myCanvas" :width="1000" :height="1000"  ref="target" @click="drawRectangle(elementX, elementY)"></canvas>
    </div>
    <button @click="advanceGrid()">Advance Grid</button>
    <button @click="playSim()">Play Simulation</button>
    <button @click="pauseSim()">Pause Simulation</button>
    <button @click="resetGrid()">Reset Grid</button>
    <button @click="setupGrid()">Setup Grid</button>
    <p>mouse_x : {{ elementX }} and mouse_y : {{ elementY }}</p>
</template>
  
<script setup>
import { onMounted } from 'vue'
import rough from 'roughjs'
import { ref } from 'vue'
import { useMouseInElement } from '@vueuse/core'


const SIZE = 20
const target = ref(null)
const { elementX, elementY } = useMouseInElement(target)
let grid = [] 
const LINES = 1000 / SIZE
const COLUMNS = 1000 / SIZE
let roughCanvas = null
let interval = null
let isPlaying = false

onMounted(() => {
    roughCanvas = rough.canvas(document.getElementById('myCanvas'))
    let generator = roughCanvas.generator;
    for (let x = 0; x < 1000; x += SIZE) {
        let line = []
        for (let y = 0; y < 1000; y += SIZE) {
            let rect = generator.rectangle(x, y, SIZE, SIZE, { fill: 'black' , fillStyle: 'solid'})         
            line.push({rect, alive: false})
        }
        grid.push(line)
    }

    setupGrid()

    for (let i = 0; i < LINES; i++) {
        for (let j = 0; j < COLUMNS; j++) {
            roughCanvas.draw(grid[i][j].rect)
        }
    }
})

function getGridRectangleFromMousePosition(x, y) {
    let i = Math.floor(x / SIZE)
    let j = Math.floor(y / SIZE)
    return {i,j}
}

function drawRectangle(x, y) {
    let {i , j} = getGridRectangleFromMousePosition(x, y)
    toggleCell(i,j)
}

function toggleCell(i, j) {
    grid[i][j].alive = !grid[i][j].alive
    if (grid[i][j].alive) {
        grid[i][j].rect.options.fill = 'red'
    } else {
        grid[i][j].rect.options.fill = 'black'
    }
    roughCanvas.draw(grid[i][j].rect)
}

function killCell(rectangle) {
    rectangle.alive = false
    rectangle.rect.options.fill = 'black'
    roughCanvas.draw(rectangle.rect)
}

function reviveCell(rectangle) {
    rectangle.alive = true
    rectangle.rect.options.fill = 'red'
    roughCanvas.draw(rectangle.rect)
}

function advanceGrid() {
    let nextGrid = JSON.parse(JSON.stringify(grid));
    for (let i=0; i<LINES; i++) {
        for (let j=0; j<COLUMNS; j++) {
            let neighbors = getNeighborsCount(i, j)
            if (grid[i][j].alive) {
                if (neighbors < 2 || neighbors > 3) {
                    killCell(nextGrid[i][j])
                }
            } else {
                if (neighbors === 3) {
                    reviveCell(nextGrid[i][j])
                }
            }
        }   
    }
    grid = nextGrid
}

function getNeighborsCount(i, j) {
    let neighbors = 0
    if (i > 0 && grid[i-1][j].alive) {
        neighbors++
    }
    if (i < LINES - 1 && grid[i+1][j].alive) {
        neighbors++
    }
    if (j > 0 && grid[i][j-1].alive) {
        neighbors++
    }
    if (j < COLUMNS - 1 && grid[i][j+1].alive) {
        neighbors++
    }
    if (i > 0 && j > 0 && grid[i-1][j-1].alive) {
        neighbors++
    }
    if (i > 0 && j < COLUMNS - 1 && grid[i-1][j+1].alive) {
        neighbors++
    }
    if (i < LINES - 1 && j > 0 && grid[i+1][j-1].alive) {
        neighbors++
    }
    if (i < LINES - 1 && j < COLUMNS - 1 && grid[i+1][j+1].alive) {
        neighbors++
    }
    return neighbors
}

function playSim() {
    if (isPlaying == false) {
        interval = setInterval(advanceGrid, 100)
    }
    isPlaying = true
}

function pauseSim() {
    clearInterval(interval)
    isPlaying = false
}

function resetGrid() {
    pauseSim()
    for (let i = 0; i < LINES; i++) {
        for (let j = 0; j < COLUMNS; j++) {
            if (grid[i][j].alive) {
                toggleCell(i, j)
            }
        }
    }
}

function setupGrid() {
    pauseSim()
    resetGrid()
    for (let i = 22; i < 28; i++) {

        toggleCell(i, 25)
        toggleCell(i, 26)
        toggleCell(i, 27)

    }
}

</script>

  
<style scoped>
.canvas-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh; 
}
</style>
  