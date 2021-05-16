<template>
  <div>
    <p class="decode-result">QRコードのデータ: <b>{{ result }}</b></p>

    <qrcode-stream :camera="camera" @decode="onDecode" @init="onInit" :track="paintOutline">
      <div v-if="validationFailure" class="validation-failure">
        URLではないQRコードです。
      </div>

      <div v-if="validationPending" class="validation-pending">
        Validation in progress...
      </div>
    </qrcode-stream>
  </div>
</template>

<script>
import { QrcodeStream } from 'vue-qrcode-reader'

export default {

  components: { QrcodeStream },

  data () {
    return {
      isValid: undefined,
      camera: 'auto',
      result: null,
    }
  },

  computed: {
    validationPending () {
      return this.isValid === undefined
          && this.camera === 'off'
    },

    validationFailure () {
      return this.isValid === false
    }
  },

  methods: {

    onInit (promise) {
      promise
          .catch(console.error)
          .then(this.resetValidationState)
    },

    resetValidationState () {
      this.isValid = undefined
    },

    async onDecode (content) {
      this.result = content
      this.turnCameraOff()

      // pretend it's taking really long
      await this.timeout(2000)
      this.isValid = content.startsWith('http')
      if ( this.isValid ) {
        window.location = content
      }

      // some more delay, so users have time to read the message
      await this.timeout(2000)

      this.turnCameraOn()
    },

    turnCameraOn () {
      this.camera = 'auto'
    },

    turnCameraOff () {
      this.camera = 'off'
    },

    timeout (ms) {
      return new Promise(resolve => {
        window.setTimeout(resolve, ms)
      })
    },

    paintOutline (detectedCodes, ctx) {
      for (const detectedCode of detectedCodes) {
        const [ firstPoint, ...otherPoints ] = detectedCode.cornerPoints

        ctx.strokeStyle = "red";

        ctx.beginPath();
        ctx.moveTo(firstPoint.x, firstPoint.y);
        for (const { x, y } of otherPoints) {
          ctx.lineTo(x, y);
        }
        ctx.lineTo(firstPoint.x, firstPoint.y);
        ctx.closePath();
        ctx.stroke();
      }
    }

  }
}
</script>

<style scoped>
.validation-failure,
.validation-pending {
  position: absolute;
  width: 100%;
  height: 100%;

  background-color: rgba(255, 255, 255, .8);
  text-align: center;
  font-weight: bold;
  font-size: 1.4rem;
  padding: 10px;

  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}
.validation-failure {
  color: red;
}
</style>
