<template>
  <div class="hello">
    <svg ref="svgCard" width="860px" height="200px" viewBox="0 0  860 200">
      <rect id="svgEditorBackground" x="0" y="0" width="860" height="200" style="fill: none; stroke: none;"/>
      <text style="fill:orangered;font-family:Arial;font-size:50px;" x="52" y="43" id="e1_texte">{{ msg }}</text>
    </svg>
    <input v-model="msg" type="text">
    <button @click="create">create</button>
  </div>
</template>

<script>
import firebase from 'firebase'

const config = {
  apiKey: "自分のapiKey",
  authDomain: "自分のauthDomain",
  databaseURL: "自分のdatabaseURL",
  projectId: "自分のprojectId",
  storageBucket: "自分のstorageBucket",
  messagingSenderId: "自分のmessagingSenderId"
};
firebase.initializeApp(config)
const db = firebase.firestore()

// svgをpngに変換
const svg2imageData = (svgElement, successCallback, errorCallback) => {
  const canvas = document.createElement('canvas')
  canvas.width = 1200
  canvas.height = 630
  const ctx = canvas.getContext('2d')
  const image = new Image()
  image.onload = () => {
    ctx.drawImage(image, 0, 0, 1200, 630)
    successCallback(canvas.toDataURL())
  }
  image.onerror = (e) => {
    errorCallback(e)
  }
  const svgData = new XMLSerializer().serializeToString(svgElement)
  image.src = 'data:image/svg+xml;charset=utf-8;base64,' + btoa(unescape(encodeURIComponent(svgData)))
}

export default {
  name: 'hello',
  data () {
    return {
      msg: 'Hello World',
      uuid: '1', // 適当に採番する
      description: 'Vue.jsとFirebaseでOGP生成アプリをつくってみます'
    }
  },
  methods: {
    async create() {
      svg2imageData(this.$refs.svgCard, async (data) => {
        const sRef = firebase.storage().ref()
        const fileRef = sRef.child(`${this.uuid}.png`)

        // Firebase Cloud Storageにアップロード
        await fileRef.putString(data, 'data_url');
        const url = await fileRef.getDownloadURL()
        console.log(url)

        // Firestoreに保存しておく
        const card = db.collection('cards').doc(this.uuid)
        await card.set({
          url,
          message: this.description
        });
      })
    }
  }
}
</script>