<template>
  <div id="app">
    <div>
      <video id="video" :width="vWidth" :height="vHeight" style="border: 1px solid gray"></video>
      <div class="h-list">
        <button @click="enableScan">扫码</button>
        <button @click="cancelScan">取消</button>
      </div>
    </div>
    <div>
      <textarea name="result" cols="50" rows="3" v-model="scanResult"></textarea>
    </div>
    <div>Fetch header</div>
    <div>
      <button @click="query">查询</button>
    </div>
    <div>
      <div v-for="(v, index) in queryResult" :key="index">
        <span>{{queryResult[index]}}</span>
        <button @click="queryResult.splice(index, 1)">删除</button>
      </div>
      <button @click="clearQuery">清空</button>
    </div>
  </div>
</template>

<script>
import { BrowserQRCodeReader } from '@zxing/library'
import { ref, computed } from 'vue'

export default {
  components: {},
  name: 'App',
  setup() {
    const scanResult = ref('')

    const qrscan = new BrowserQRCodeReader()

    // const imageUrl = ref('')

    // const imageAdded = (e) => {
    //   if (e.target.files.length == 0) {
    //     return
    //   }
    //   let file = e.target.files[0]
    //   let tempImage = new Image()

    //   tempImage.src = URL.createObjectURL(file)
    //   tempImage.onload = () => {
    //     console.log(tempImage.naturalWidth)
    //     console.log(tempImage.naturalHeight)
    //     let width = Math.min(tempImage.naturalWidth, 300)
    //     let height = (tempImage.naturalHeight / tempImage.naturalWidth) * width
    //     console.log(width)
    //     console.log(height)

    //     let canvas = document.createElement('canvas')
    //     canvas.width = width
    //     canvas.height = height

    //     canvas.getContext('2d').drawImage(tempImage, 0, 0, width, height)
    //     imageUrl.value = canvas.toDataURL('image/jpeg')
    //     URL.revokeObjectURL(tempImage.src)
    //   }
    // }

    let vWidth = ref(300)
    let vHeight = ref(200)
    const enableScan = async () => {
      const media = await navigator.mediaDevices
        .getUserMedia({
          video: {
            facingMode: 'environment',
          },
        })
        .then((media) => {
          let { width, height } = media.getTracks()[0].getSettings()
          let newHeight = Math.min(height, 500)
          let newWidth = (width / height) * newHeight
          vWidth.value = newWidth
          vHeight.value = newHeight
          return media
        })
      qrscan.decodeOnceFromStream(media, 'video').then((value) => {
        // qrscan.decodeFromImageUrl(imageUrl.value).then((value) => {
        scanResult.value = value
        qrscan.stopStreams()
      })
    }

    const queryResult = ref([])

    const query = async () => {
      let string = scanResult.value.toString()
      if (string && string.lastIndexOf('#') > 0) {
        let code = string.substring(string.lastIndexOf('#') + 1)
        let res = await fetch(
          'https://cors.unluckyninja.workers.dev/?http://wxcx.mem.gov.cn/certsearch/cert/scanQrcode',
          {
            method: 'POST',
            headers: {
              credentials: 'include',
              'Content-Type': 'application/json',
            },
            body: JSON.stringify({ qrcode: code }),
          }
        )

        queryResult.value.push((await res.json()).content.certNum)
      }
    }

    return {
      vWidth,
      vHeight,
      // imageUrl,
      // imageAdded,
      qrscan,
      enableScan,
      scanResult,
      // get info
      queryResult,
      query,
    }
  },

  methods: {
    saveHost(option) {
      if (option == 'https') {
        window.localStorage.setItem('httpsHost', this.httpsHost)
        this.httpsJumpLink = 'https://' + this.httpsHost
      } else {
        window.localStorage.setItem('httpHost', this.httpHost)
        this.httpJumpLink =
          'http://' +
          this.httpHost +
          '/?code=' +
          encodeURIComponent(this.scanResult)
      }
    },
    clearQuery(){
      this.queryResult.length = 0
    },
    cancelScan(){
      this.qrscan.stopStreams()
    }
  },
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  & *{
    font-size: 1.5rem;
  }
}
div {
  padding: 4px;
}
button {
  margin: 8px;
  user-select: none;
  font-size: 1.2rem
}
.h-list{
  display: flex;
  justify-content: space-around;
}
</style>
