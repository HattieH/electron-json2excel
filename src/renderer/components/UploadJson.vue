<template>
  <div>
    <el-button
      :loading="loading"
      size="mini" 
      type="primary"
      @click='handleUpload'>
      {{text}}
    </el-button>
    <!-- <slot v-if='drop'></slot> -->
    <input ref='upload-json-input'
     type='file' 
     class='upload-json-input'
     @change='handleChange'>
  </div>
</template>
<script>
const fs = require('fs')

export default {
  name: 'hs-upload-json',
  props: {
    text: {
    type: String,
    default: 'Browse'
    },
    accepts: {
      type: String,
      default: '.xlsx, .xls'
    },
    beforeUpload: Function,
    uploadSuccess: Function
  },
  data() {
      return {
        loading: false,
        excelData: {
            header: null,
            results: null
        }
      }
  },
  methods: {
    handleUpload() {
      this.$refs['upload-json-input'].click();
    },
    handleChange(e) {
      const files = e.target.files;
      console.log(e)
      const rawFile = files[0];  // only use files[0]
      if (!rawFile) return
      this.upload(rawFile)
    },
    upload(rawFile) {
      this.$refs['upload-json-input'].value = null;

      if (!this.beforeUpload) {
        this.readData(rawFile);
        return;
      }

      const before = this.beforeUpload(rawFile);
      if (before && before.then) {
        before.then(processedFile => {
          this.readData(processedFile);
        })
        .catch((err) => {
          console.log(err);
          return
        }) 
      } else if (before === false) {
        return;
      } else {
        this.readData(rawFile);
      }
    },
    readData(file) {
      this.loading = true;
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = e => {
          let data = e.target.result;
          this.generateData(data);
          this.loading = false;
          resolve();
        };
        reader.readAsText(file)
      })
    },
    generateData(data) {
      this.uploadSuccess && this.uploadSuccess(data);
    }
  }
}
</script>
<style lang="scss" scoped>
.upload-json-input{
  display: none;
  z-index: -9999;
}
</style>
