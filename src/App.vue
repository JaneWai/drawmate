<template>
  <Whiteboard />
</template>

<script setup lang="ts">
import Whiteboard from './components/Whiteboard.vue'
</script>

<style>
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

#app {
  width: 100vw;
  height: 100vh;
  max-width: 100%;
  margin: 0;
  padding: 0;
}
</style>
