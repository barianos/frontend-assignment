<template>
  <div id="app">
    <HereMap :center="center"/>
  </div>
</template>

<script>
import HereMap from "./components/HereMaps";
export default {
  name: "app",
  components: {
    HereMap,
  },
  data() {
    return {
      center: {
        lat: 37.1666666,
        lng: 24.935242,
      },
    };
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
