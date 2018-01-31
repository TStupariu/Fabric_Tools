<template>
  <div id="app">
  	<canvascomponent></canvascomponent>
  </div>
</template>

<script>
import canvascomponent from './components/canvascomponent.vue'

export default {
  name: 'app',
  components: {
    canvascomponent
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
