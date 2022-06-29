<template>
  <heat-map />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import HeatMap from "./components/heat-map.vue";

export default defineComponent({
  name: "App",
  components: { HeatMap },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 30vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
