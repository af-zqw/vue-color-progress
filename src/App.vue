<template>
  <div id="app">
    <Main type="square" :progress="50"></Main>
  </div>
</template>

<script>
import Main from './components/Main'
export default {
  name: 'app',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  components: {
    Main
  }
}
</script>

<style>
</style>
