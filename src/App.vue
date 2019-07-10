<template>
  <div id="app">
    <h1>GoJS + Vue.js + Webpack:</h1>
    <Diagram></Diagram>
  </div>
</template>

<script>
import Diagram from "./components/Diagram";

export default {
  name: "App",
  components: {
    Diagram
  }
};
</script>


<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 10px;
}
</style>
