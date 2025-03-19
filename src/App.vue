<template>
  <NavBar/>
  <LineupBuilder/>
</template>

<script>

import LineupBuilder from './components/LineupBuilder.vue';
import NavBar from './components/NavBar.vue';

export default {
  name: 'App',
  components: {
    LineupBuilder,
    NavBar

  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
