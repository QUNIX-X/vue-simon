<template>
  <div id="app">
    <Simon />
  </div>
</template>

<script>
import Simon from "@/components/Simon";

export default {
  name: "App",

  components: {
    Simon,
  },
};
</script>

<style lang="scss">
#app {
  font-family: "Comfortaa", Helvetica, Arial, sans-serif;
  font-weight: 900;
  letter-spacing: 4px;
}
</style>
