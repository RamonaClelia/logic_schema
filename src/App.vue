<template>
  <div id="app">
    <SelectionGrid2 msg="Welcome to Your Vue.js App"/>    
  </div>
</template>

<script>
import SVG from "svg.js";
import SelectionGrid2 from './components/SelectionGrid2.vue'

export default {
  name: 'app',
  components: {
    SelectionGrid2,    
  },
  
}
</script>

<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  font-family: cursive;
}
</style>
