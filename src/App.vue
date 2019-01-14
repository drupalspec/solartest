<template>
  <div id="app">
    <main-view/>
  </div>
</template>

<script>
import mainView from './components/MainView.vue'

export default {
  name: 'app',
  components: {
    mainView
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
