<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png" />
    <QrReader msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
// @ is an alias to /src
import QrReader from "@/components/QrReader.vue";

export default {
  name: "home",
  components: {
    QrReader
  }
};
</script>
