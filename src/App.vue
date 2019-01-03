<template>
  <div id="app">
    <weather/>
  </div>
</template>

<script>
import Weather from './components/weather.vue'

export default {
  name: 'app',
  components: {
    Weather
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  max-width: 480px;
  margin: 60px auto 0;
}
</style>
