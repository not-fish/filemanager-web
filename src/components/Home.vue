<template>
  <div>
    <a-divider>文件上传</a-divider>
    <a-row>
      <a-col :span="12" :offset="6">
        <UploadFile></UploadFile>
      </a-col>
    </a-row>

    <a-divider>文件列表</a-divider>
    <a-row>
      <a-col :span="20" :offset="2">
        <FileList></FileList>
      </a-col>
    </a-row>
  </div>
</template>

<script>

  import UploadFile from "./UploadFile";
  import FileList from "./FileList";

  export default {
    name: 'Home',
    components:{
      UploadFile,FileList
    },
    data () {
      return {
        msg: 'Welcome to Your Vue.js App'
      }
    },
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
