<template>
  <div id="app">
    <header class="header">
      <h1>Perspective + Apache Arrow Demo</h1>
    </header>
    <PerspectiveWrapper />
  </div>
</template>

<script>
import PerspectiveWrapper from './components/PerspectiveWrapper.vue'

export default {
  name: 'App',
  components: {
    PerspectiveWrapper
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', 'Helvetica', 'Arial', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.header {
  margin-bottom: 16px;
}

.header h1 {
  color: #1a73e8;
  margin-bottom: 8px;
}
</style>