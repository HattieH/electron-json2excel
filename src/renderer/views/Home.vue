<template>
  <div>
    <el-row :gutter='24'>
      <el-col :span='8'>
        <upload-json 
          :uploadSuccess='uploadOldJson'
          style='margin:10px 0'>上传</upload-json>
        <el-input
          type='textarea'
          v-model='oldJsonData'
          :rows='30'></el-input>
      </el-col>
      <el-col :span='8'>
        <upload-json 
          :uploadSuccess='uploadNewJson'
          style='margin:10px 0'>上传</upload-json>
        <el-input
          type='textarea'
          v-model='newJsonData'
          :rows='30'></el-input>
      </el-col>
      <el-col :span='8'>
        <el-button-group>
          <el-button
            type='primary'
            style='margin:10px 0'
            @click='previewAdd'>
            预览增量
          </el-button>
          <el-button
            type='primary'
            style='margin:10px 0'
            @click='generateFile'>
            生成excel文件
          </el-button>
        </el-button-group>
        <el-input
          type='textarea'
          v-model='addJsonData'
          :rows='30'></el-input>
      </el-col>
    </el-row>
    <el-row>
      <upload-excel 
        text='Excel2Json'
        :uploadSuccess='importEXCEL2JSONFile'
        style='margin:10px 0'></upload-excel>
    </el-row>
  </div>
</template>
<script>
import uploadJson from '@/components/UploadJson'
import export2Excel from '@/utils/Export2Excel'
import uploadExcel from '@/components/UploadExcel'

export default {
  name: 'home',
  data() {
    return {
      oldJsonData: '',
      newJsonData: '',
      addJsonData: null
    }
  },
  components: {
    uploadJson,
    uploadExcel
  },
  methods: {
    uploadOldJson(data) {
      this.oldJsonData = data;
    },
    uploadNewJson(data) {
      this.newJsonData = data;
    },
    previewAdd() {
      const olds = JSON.parse(this.oldJsonData)
      const news = JSON.parse(this.newJsonData)
      console.log(olds)
      let tmp = {};
      for (let key in news) {
        console.log(key)
        if (!olds[key]) {
          tmp[key] = news[key];
        } else if (olds[key] !== news[key]) {
          tmp[key] = news[key];
        }
      }
      this.addJsonData = JSON.stringify(Object.assign(tmp), null, 4);
    },
    generateFile() {
      const tHeader = ["key", "英文", "翻译"];
      const obj = JSON.parse(this.addJsonData);
      let data = [];
      console.log(obj)
      for (let key in obj) {
        data.push([key, obj[key]])
      }
      console.log(data)
      import('@/utils/Export2Excel').then(excel => {
        excel.export_json_to_excel({
            header: tHeader, //表头 必填
            data, //具体数据 必填
            filename: '新建excel', //非必填
            autoWidth: true, //非必填
            bookType: 'xlsx' //非必填
        })
      })
    },
    importEXCEL2JSONFile(data) {
      console.log(data)
      let jsonObj = {}
      for (let i = 0, len = data.results.length; i < len; i++) {
        let keyObj = data.results[i]
        jsonObj[keyObj['key']] = keyObj['翻译']
      }
      console.log(jsonObj)
      const blob = new Blob([JSON.stringify(jsonObj, null, 4)], {type: ''})
      saveAs(blob, 'hahaha.json')
    }
  }
}
</script>
<style lang="scss" scoped>

</style>
