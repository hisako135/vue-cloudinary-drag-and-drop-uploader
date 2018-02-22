<template>
  <div>
    <h2>Drug & Drop File Uploader</h2>
    <!-- <input type="file" v-on:change="upload($event.target.files)" accept="image/*"> -->
    <div id="drop-area" @dragover.prevent @dragover="hovering = true" @dragleave="hovering = false" :class="{'highlight': hovering}">
        <form class="my-form" ref="fileform">
            <p>Upload multiple files with the file dialog or by dragging and dropping images onto the dashed region</p>
            <input type="file" id="fileElem" v-on:change="upload($event.target.files)" multiple accept="image/*">
            <label class="button" for="fileElem" >Select some files</label>
            <progress id="progress-bar" max=100 value=0></progress>
            <div id="gallery">
              <div v-for="file in files" :key="file.key" class="flie-listening">
                <img class="preview" :ref="'preview' + parseInt(file.key)">
                {{ file.name }}
              </div>
              <!-- <div v-for="(file, id) in files" :key="file.key" class="file-listing">
                  <img class="preview" v-bind:ref="'preview'+parseInt(id)"/>
                  {{ file.name }}
              </div> -->
            </div>
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
      dragAndDropCpable: false,
      files: [],
      hovering: false,
      cloudinary: {
        uploadPreset: config.CLD_UPLOAD_PRESET,
        apiKey: config.CLD_API_KEY,
        cloudName: config.CLD_CLOUD_NAME
      }
    }
  },
  computed: {
    cldUrl: function () {
      return `https://api.cloudinary.com/v1_1/${this.cloudinary.cloudName}/image/upload`
    }
  },
  mounted () {
    this.dragAndDropCpable = this.determineDragAndDropCapable()
    console.log(this.dragAndDropCpable)

    if (this.dragAndDropCpable) {
      /*
        fileform内の全てのドラッグイベントのデフォルトイベントを抑止
      */
      ['drag', 'dragstart', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach(evt => {
        this.$refs.fileform.addEventListener(evt, function (e) {
          e.preventDefault()
          e.stopPropagation()
        }, false)
      })
      /*
        fileformにドロップイベントを登録
      */
      this.$refs.fileform.addEventListener('drop', function (e) {
        for (let i = 0; i < e.dataTransfer.files.length; i++) {
          this.files.push(e.dataTransfer.files[i])
          this.getImagePreviews()
        }

        this.uploadFiles(this.files)
      }.bind(this))
    }
  },
  methods: {
    /*
      DnD可能なブラウザか判定
    */
    determineDragAndDropCapable () {
      const div = document.createElement('div')
      return (('draggable' in div) || ('ondragstart' in div && 'ondrop' in div)) && 'FormData' in window && 'FileReader' in window
    },
    /*
      プレビュー
    */
    getImagePreviews () {
      for (let i = 0; i < this.files.length; i++) {
        if (/\.(jpe?g|png|gif)$/i.test(this.files[i].name)) {
          let reader = new FileReader()

          reader.addEventListener('load', function () {
            console.log(this.files[i].name)
            console.log(this.$refs)
            this.$refs['preview' + parseInt(i)][0].src = reader.result
          }.bind(this), false)

          reader.readAsDataURL(this.files[i])
        } else {
          // this.$nextTick(() => {
          //   this.$refs['preview' + parseInt(i)][0].src = '/images/file.png'
          // })
          console.log('画像じゃないお')
        }
      }
    },
    /*
      アップロードしてAPI通信
    */
    uploadFiles () {
      let formData = new FormData()

      for (let i = 0; i < this.files.length; i++) {
        let file = this.files
        formData.append('file', file[i])
        formData.append('upload_preset', config.CLD_UPLOAD_PRESET)
        formData.append('tags', 'gs-vue,gs-vue-uploaded')
      }

      // API通信
      axios.post(this.cldUrl, formData)
        .then(res => { console.log('Success！') })
        .catch(() => { console.error() })
    }
  //   // ファイルドロップ時の処理
  //   handleDrop: function (e) {
  //     console.log('どろっぷ！')
  //     let dt = e.dataTransfer
  //     let files = dt.files

  //     this.handleFiles(files)
  //   },
  //   // ファイルドロップ後の処理
  //   handleFiles: function (files) {
  //     files = [...files]
  //     this.initializeProgress(files.length)
  //     files.forEach(this.uploadFile)
  //     files.forEach(this.previewFile)
  //   },
  //   // プレビュー
  //   previewFile: function (files) {
  //     let reader = new FileReader()
  //     reader.readAsDataURL(files)
  //     reader.onloadend = function () {
  //       let img = document.createElement('img')
  //       img.src = reader.result
  //       document.getElementById('gallery').appendChild(img)
  //     }
  //   },
  //   // プログレスバー
  //   initializeProgress: function (numfiles) {
  //     this.progressBar.value = 0
  //     this.filesDone = 0
  //     this.filesToDo = numfiles
  //   },
  //   progressDone: function () {
  //     this.filesDone++
  //     this.progressBar.value = this.filesDone / this.filesToDo * 100
  //   },
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

.file-listing{
  width: 400px;
  margin: auto;
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

.file-listing img{
  height: 100px;
}
</style>
