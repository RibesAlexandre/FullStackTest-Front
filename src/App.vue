<template>
  <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
  <ExchangeTableRate />
  </div>
</template>

<script>
import ExchangeTableRate from "./components/ExchangeTableRate.vue";

export default {
  name: 'App',
  components: {
    ExchangeTableRate,
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
