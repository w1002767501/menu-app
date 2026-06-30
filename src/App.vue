<template>
  <div id="app">
    <MenuApp />
  </div>
</template>

<script>
import MenuApp from './components/MenuApp.vue'

export default {
  name: 'App',
  components: { MenuApp }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  background: #FEF7ED;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>