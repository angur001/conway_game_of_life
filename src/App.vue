<template>
  <div>
    <button @click="showGrid = !showGrid">Toggle Grid</button>
    <grid v-if="showGrid"/>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import grid from './components/grid.vue'

let showGrid = ref(true)
</script>
