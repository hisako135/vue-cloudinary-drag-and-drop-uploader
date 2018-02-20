<template>
  <div>
    <h2>{{ msg }}</h2>
    <input type="file" v-on:change="upload($event.target.files)" accept="image/*">
  </div>
</template>

<script>
import config from '../../.cld.json'
import axios from 'axios'

export default {
  data () {
    return {
      cloudinary: {
        uploadPreset: config.CLD_UPLOAD_PRESET,
        apiKey: config.CLD_API_KEY,
        cloudName: config.CLD_CLOUD_NAME
      },
      msg: 'Drug & Drop File Uploader'
    }
  },
  computed: {
    cldUrl: function () {
      return `https://api.cloudinary.com/v1_1/${this.cloudinary.cloudName}/image/upload`
    }
  },
  methods: {
    // アップロード時の処理
    upload: function (file) {
      const formData = new FormData()
      formData.append('file', file[0])
      formData.append('upload_preset', config.CLD_UPLOAD_PRESET)
      formData.append('tags', 'gs-vue,gs-vue-uploaded')
      // For debug purpose only
      // Inspects the content of formData
      for (let pair of formData.entries()) {
        console.log(pair[0] + ', ' + pair[1])
      }
      console.log(this.cldUrl)
      // Execute api
      axios.post(this.cldUrl, formData)
        .then(res => { console.log('Success！') })
        .catch(() => { console.error() })
    }
  }
}
</script>

<style scoped>
#drop-area {
    border: 2px dashed #ccc;
    border-radius: 20px;
    width: 480px;
    font-family: sans-serif;
    margin: 100px auto;
    padding: 20px;
}
#drop-area.highlight {
    border-color: purple;
}
p {
    margin-top: 0;
}
.my-form {
    margin-bottom: 10px;
}
#gallery {
    margin-top: 10px;
}
#gallery img {
    width: 150px;
    margin-bottom: 10px;
    margin-right: 10px;
    vertical-align: middle;
}
.button {
    display: inline-block;
    padding: 10px;
    background: #ccc;
    cursor: pointer;
    border-radius: 5px;
    border: 1px solid #ccc;
}
.button:hover {
    background: #ddd;
}
#fileElem {
    display: none;
}
</style>
