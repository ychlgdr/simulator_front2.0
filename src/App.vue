<template>
  <div id="app">
    <graphEditor/>
  </div>
</template>

<script>
import graphEditor from './components/GraphEditor.vue'

export default {
  name: 'App',
  components: {
    graphEditor
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
}
</style>
