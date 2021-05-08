<template>
  <v-app id="app">
    <CurrencySheet/>
  </v-app>
</template>

<script>
import CurrencySheet from "@/components/CurrencySheet";

export default {
  name: 'App',
  components: {
    CurrencySheet,
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
}
</style>
