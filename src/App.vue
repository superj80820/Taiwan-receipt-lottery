<template>
  <v-app>
    <v-app-bar app>
      <v-toolbar-title class="headline">
        <span style="font-size:20px;">請掃描發票左側QR code</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        text
        href="https://github.com/superj80820/Taiwan-receipt-lottery"
        target="_blank"
      >
        <span class="mr-2">使用方法</span>
      </v-btn>
    </v-app-bar>

    <v-content>
      <QrReader />
    </v-content>
  </v-app>
</template>

<script>
import QrReader from "./components/QrReader";

export default {
  name: "App",
  components: {
    QrReader
  },
  data: () => ({
    //
  })
};
</script>
