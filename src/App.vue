<template>
  <div id="app">
    <div class="container">
      <GrowthCalculator/>
    </div>
  </div>
</template>

<script>
import GrowthCalculator from './components/GrowthCalculator.vue'

export default {
  name: 'app',
  components: {
    GrowthCalculator
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
