<template>
  <div>
    <h2>{{ msg }}</h2>
    <!-- <input type="file" v-on:change="upload($event.target.files)" accept="image/*"> -->
    <div id="drop-area">
        <form class="my-form">
            <p>Upload multiple files with the file dialog or by dragging and dropping images onto the dashed region</p>
            <input type="file" id="fileElem" v-on:change="upload($event.target.files)" multiple accept="image/*">
            <label class="button" for="fileElem" >Select some files</label>
            <progress id="progress-bar" max=100 value=0></progress>
            <div id="gallery"></div>
        </form>
    </div>
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
  created: function () {
    const dropArea = document.getElementById('drop-area')
    const progressBar = document.getElementById('progress-bar')
    let filesToDo = 0
    let filesDone = 0

    ;['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
      dropArea.addEventListener(eventName, highlight, false)
    })
    ;['dragenter', 'dragover'].forEach(eventName => {
      dropArea.addEventListener(eventName, highlight, false)
    })
    ;['dragleave', 'drop'].forEach(eventName => {
      dropArea.addEventListener(eventName, unhighlight, false)
    })

    dropArea.addEventListener('drop', handleDrop, false)
  },
  methods: {
    // デフォルトのイベントをキャンセル
    preventDefaults: function (e) {
      e.preventDefault()
      e.stopPropagation()
    },
    // ドラッグオーバー時のハイライト
    highlight: function (e) {
      console.log('どらっぐおーばー！')
      dropArea.classList.add('highlight')
    },
    // ドラッグアウト時はハイライトしない
    unhighlight: function (e) {
      console.log('どらっぐあうと！')
      dropArea.classList.remove('highlight')
    },
    // ファイルドロップ時の処理
    handleDrop: function (e) {
      console.log('どろっぷ！')
      let dt = e.dataTransfer
      let files = dt.files

      handleFiles(files)
    },
    // ファイルドロップ後の処理
    handleFiles: function (files) {
      files = [...files]
      initializeProgress(files.length)
      files.forEach(uploadFile)
      files.forEach(previewFile)
    },
    // プレビュー
    previewFile: function (files) {
      let reader = new FileReader()
      reader.readAsDataURL(file)
      reader.onloadend = function () {
        let img = document.createElement('img')
        img.src = reader.result
        document.getElementById('gallery').appendChild(img)
      }
    },
    // プログレスバー
    initializeProgress: function (numfiles) {
      progressBar.value = 0
      filesDone = 0
      filesToDo = numfiles
    },
    progressDone: function () {
      filesDone++
      progressBar.value = filesDone / filesToDo * 100
    },
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
    margin: 0 auto;
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
