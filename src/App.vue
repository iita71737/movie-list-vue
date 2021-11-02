<template>
  <div id="app">
    <Movielist />
  </div>
</template>

<script>
import "bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";
import Movielist from "./components/Movielist.vue";

export default {
  name: "App",
  components: {
    Movielist,
  },
};
</script>
