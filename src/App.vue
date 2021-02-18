<template>
  <div id="app">
    <Fund />
  </div>
</template>

<script>
import Fund from './components/Fund'

export default {
  name: 'App',
  components: {
    Fund
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
