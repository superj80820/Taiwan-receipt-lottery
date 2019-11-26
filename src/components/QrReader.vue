<template>
  <v-container>
    <v-layout wrap>
      <div class="text-center mb-4">
        <v-overlay :opacity="0.5" z-index="1" :value="alert">
          <v-alert color="green" dark transition="scale-transition">
            {{ result }}
          </v-alert>
        </v-overlay>
      </div>
      <v-card class="mx-auto" outlined>
        <v-radio-group style="padding-left: 15px" v-model="radios" row>
          <v-radio label="自動掃描" value="QRMode"></v-radio>
          <v-radio label="手動輸入" value="manual"></v-radio>
        </v-radio-group>
        <v-list-item>
          <v-list-item-content v-show="radios === 'manual'">
            <v-text-field
              outlined
              v-model="manualInput"
              label="手動輸入"
            ></v-text-field>
          </v-list-item-content>
          <v-list-item-content v-show="radios === 'QRMode'">
            <!-- <qrcode-stream @decode="onDecode" @init="onInit" /> -->
            <video ref="preview"></video>
          </v-list-item-content>
        </v-list-item>
        <v-card-title class="headline"
          >特別獎 {{ winningNum.special.join(" ") }}</v-card-title
        >
        <v-card-subtitle
          >同期統一發票收執聯8位數號碼與特別獎號碼相同者獎金1,000萬元</v-card-subtitle
        >
        <v-card-title class="headline"
          >特獎 {{ winningNum.grand.join(" ") }}</v-card-title
        >
        <v-card-subtitle
          >同期統一發票收執聯8位數號碼與特獎號碼相同者獎金200萬元</v-card-subtitle
        >
        <v-card-title class="headline"
          >頭獎 {{ winningNum.first.join(" ") }}</v-card-title
        >
        <v-card-subtitle
          >同期統一發票收執聯8位數號碼與頭獎號碼相同者獎金20萬元</v-card-subtitle
        >
        <v-card-title class="headline">二獎</v-card-title>
        <v-card-subtitle
          >同期統一發票收執聯末7 位數號碼與頭獎中獎號碼末7
          位相同者各得獎金4萬元</v-card-subtitle
        >
        <v-card-title class="headline">三獎</v-card-title>
        <v-card-subtitle
          >同期統一發票收執聯末6 位數號碼與頭獎中獎號碼末6
          位相同者各得獎金1萬元</v-card-subtitle
        >
        <v-card-title class="headline">四獎</v-card-title>
        <v-card-subtitle
          >同期統一發票收執聯末5 位數號碼與頭獎中獎號碼末5
          位相同者各得獎金4千元</v-card-subtitle
        >
        <v-card-title class="headline">五獎 </v-card-title>
        <v-card-subtitle
          >同期統一發票收執聯末4 位數號碼與頭獎中獎號碼末4
          位相同者各得獎金1千元</v-card-subtitle
        >
        <v-card-title class="headline">六獎</v-card-title>
        <v-card-subtitle
          >同期統一發票收執聯末3 位數號碼與 頭獎中獎號碼末3
          位相同者各得獎金2百元</v-card-subtitle
        >
        <v-card-title class="headline">增開六獎 </v-card-title>
        <v-card-subtitle
          >同期統一發票收執聯末3位數號碼與增開六獎號碼相同者各得獎金2百元</v-card-subtitle
        >
      </v-card>
    </v-layout>
  </v-container>
</template>

<script>
// import QrScanner from 'qr-scanner';
import winningNum from "../../public/winningNum.json";
const Instascan = require("instascan");
const space = require("../../public/sounds/征服宇宙.mp3");
const getMomey = require("../../public/sounds/發大財.mp3");

export default {
  // components: { QrcodeStream },
  data() {
    return {
      result: "",
      error: "",
      winningNum: winningNum,
      alert: false,
      radios: "QRMode",
      manualInput: "",
      sounds: {
        space: space,
        getMomey: getMomey
      }
    };
  },
  watch: {
    result() {
      if (this.result !== "") {
        let audio = new Audio(this.sounds.getMomey);
        audio.play();
        this.alert = true;
        setTimeout(() => {
          this.result = "";
          this.alert = false;
        }, 700);
      }
    },
    radios() {
      let audio = new Audio(this.sounds.space);
      audio.play();
    },
    manualInput() {
      this.checkWinning("manual", this.manualInput);
    }
  },
  mounted() {
    let scanner = new Instascan.Scanner({
      video: this.$refs.preview,
      mirror: false,
      refractoryPeriod: 100,
      scanPeriod: 1
    });
    scanner.addListener("scan", content => {
      if (this.checkNumFormat(content) && this.alert !== true) {
        this.checkWinning("QRMode", this.getNum(content));
      } else {
        // this.result = '請對準發票左側QR code'
      }
    });
    Instascan.Camera.getCameras()
      .then(function(cameras) {
        if (cameras.length >= 2) {
          scanner.start(cameras[1]);
        } else if (cameras.length === 1) {
          scanner.start(cameras[0]);
        } else {
          console.error("No cameras found.");
        }
      })
      .catch(function(e) {
        console.error(e);
      });
  },
  methods: {
    onDecode(result) {
      this.checkWinning("QRMode", this.getNum(result));
    },
    checkNumFormat(result) {
      return result.match(/^[A-Za-z]+(\d{8})/) !== null;
    },
    getNum(result) {
      return result.match(/^[A-Za-z]+(\d{8})/)[1];
    },
    checkWinning: (function() {
      function checkIncloude(self, winning, number, pattern) {
        return self.winningNum[winning]
          .map(item => item.match(pattern)[0])
          .includes(number.match(pattern)[0]);
      }
      return function(mode, number) {
        if (mode === "QRMode") {
          if (checkIncloude(this, "special", number, /\d{8}$/)) {
            this.result = `${number} 中特別獎`;
          } else if (checkIncloude(this, "grand", number, /\d{8}$/)) {
            this.result = `${number} 中特獎`;
          } else if (checkIncloude(this, "first", number, /\d{3}$/)) {
            this.result = `${number} 中六獎`;
            if (checkIncloude(this, "first", number, /\d{4}$/)) {
              this.result = `${number} 中五獎`;
              if (checkIncloude(this, "first", number, /\d{5}$/)) {
                this.result = `${number} 中四獎`;
                if (checkIncloude(this, "first", number, /\d{6}$/)) {
                  this.result = `${number} 中三獎`;
                  if (checkIncloude(this, "first", number, /\d{7}$/)) {
                    this.result = `${number} 中二獎`;
                    if (checkIncloude(this, "first", number, /\d{8}$/)) {
                      this.result = `${number} 中頭獎`;
                    }
                  }
                }
              }
            }
          } else if (checkIncloude(this, "additional", number, /\d{3}$/)) {
            this.result = `${number} 中增開六獎`;
          } else {
            this.result = `${number} 悲劇 什麼都沒中`;
          }
        } else if (mode === "manual") {
          if (number.length < 3) return false;
          if (checkIncloude(this, "special", number, /\d{3}$/)) {
            this.result = "注意特別獎";
          } else if (checkIncloude(this, "grand", number, /\d{3}$/)) {
            this.result = "注意特獎";
          } else if (checkIncloude(this, "first", number, /\d{3}$/)) {
            this.result = "中兩百注意頭獎";
          } else {
            this.result = "悲劇 什麼都沒中";
          }
        }
      };
    })()
    // async onInit(promise) {
    //   try {
    //     await promise;
    //   } catch (error) {
    //     if (error.name === "NotAllowedError") {
    //       this.error = "ERROR: you need to grant camera access permisson";
    //     } else if (error.name === "NotFoundError") {
    //       this.error = "ERROR: no camera on this device";
    //     } else if (error.name === "NotSupportedError") {
    //       this.error = "ERROR: secure context required (HTTPS, localhost)";
    //     } else if (error.name === "NotReadableError") {
    //       this.error = "ERROR: is the camera already in use?";
    //     } else if (error.name === "OverconstrainedError") {
    //       this.error = "ERROR: installed cameras are not suitable";
    //     } else if (error.name === "StreamApiNotSupportedError") {
    //       this.error = "ERROR: Stream API is not supported in this browser";
    //     }
    //   }
    // }
  }
};
</script>
