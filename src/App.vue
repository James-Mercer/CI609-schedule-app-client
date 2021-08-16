<template>
  <v-app id="app">
    <router-view />
  </v-app>
</template>

<script>
import Vue from "vue";

export default new Vue({
  name: "App",

  data: () => ({
    //
  }),
});
</script>

<style>
#app {
  width: 100vw;
  height: 100vh;
  overflow-x: hidden;
  overflow-y: scroll;
}
</style>
