<template>
  <div id="app">
    <Main :progress="50" label="msg" rate="slow"></Main>
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
