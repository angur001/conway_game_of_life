<template>
    <div class="canvas-container">
      <canvas id="myCanvas" :width="1000" :height="1000"></canvas>
    </div>
    <button @click="advanceGrid()">Advance Grid</button>
    <button @click="playSim()">Play Simulation</button>
    <button @click="pauseSim()">Pause Simulation</button>
</template>
  
  <script setup>
  import { onMounted } from 'vue'
  import rough from 'roughjs'

  const SIZE = 20

  let grid = [] 
  const LINES = 1000 / SIZE
  const COLUMNS = 1000 / SIZE
  let roughCanvas = null
  
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

    for (let i = 22; i < 28; i++) {

            toggleCell(i, 25)
            toggleCell(i, 26)
            toggleCell(i, 27)

    }

    for (let i = 0; i < LINES; i++) {
        for (let j = 0; j < COLUMNS; j++) {
            roughCanvas.draw(grid[i][j].rect)
        }
    }
  })

  function toggleCell(i, j) {
    grid[i][j].alive = !grid[i][j].alive
    if (grid[i][j].alive) {
        grid[i][j].rect.options.fill = 'red'
    } else {
        grid[i][j].rect.options.fill = 'black'
    }
    roughCanvas.draw(grid[i][j].rect)
  }

    function advanceGrid() {
        for (let i=0; i<LINES; i++) {
            for (let j=0; j<COLUMNS; j++) {
                let neighbors = getNeighborsCount(i, j)
                if (grid[i][j].alive) {
                    if (neighbors < 2 || neighbors > 3) {
                        toggleCell(i, j)
                    }
                } else {
                    if (neighbors === 3) {
                        toggleCell(i, j)
                    }
                }
                

            }   
        }
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

    let interval = null

    function playSim() {
        interval = setInterval(advanceGrid, 100)
    }

    function pauseSim() {
        clearInterval(interval)
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
  