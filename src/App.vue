<template>
  <div id="app">
    <MineZone />
  </div>
</template>

<script>
import MineZone from './components/MineZone.vue'

export default {
  name: 'App',
  components: {
    MineZone
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
