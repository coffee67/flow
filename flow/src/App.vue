<template>
  <div id="app">
    <flowMain/>
  </div>
</template>

<script>
import flowMain from './components/flowMain'

export default {
  name: 'App',
  components: {
    flowMain
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  /* margin-top: 60px; */
}
</style>
