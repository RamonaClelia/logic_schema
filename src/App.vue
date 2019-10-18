<template>
  <div id="app">
    <SelectionGrid msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import SVG from "svg.js";
import SelectionGrid from './components/SelectionGrid.vue'

export default {
  name: 'app',
  components: {
    SelectionGrid
  },
   watch: {},
   mounted() {},
    methods: {}
}
</script>

<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
