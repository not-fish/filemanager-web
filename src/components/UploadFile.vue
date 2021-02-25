<template>
  <div class="clearfix">
    <a-upload :file-list="fileList" :remove="handleRemove" :before-upload="beforeUpload" :multiple="multipleSelect">
      <a-button> <a-icon type="upload" /> 选择文件 </a-button>
    </a-upload>
    <a-button
      type="primary"
      :disabled="fileList.length === 0"
      :loading="uploading"
      style="margin-top: 16px"
      @click="handleUpload"
    >
      {{ uploading ? 'Uploading' : 'Start Upload' }}
    </a-button>
  </div>
</template>

<script>

  import axios from 'axios';

  export default {
    name: "UploadFile",
    data() {
      return {
        fileList: [],
        uploading: false,
        multipleSelect:true,
        url:{
          upload: "http://localhost:9090/filemanager/file/upload",
        },
      };
    },
    methods: {
      handleRemove(file) {
        const index = this.fileList.indexOf(file);
        const newFileList = this.fileList.slice();
        newFileList.splice(index, 1);
        this.fileList = newFileList;
      },
      beforeUpload(file) {
        this.fileList = [...this.fileList, file];
        return false;
      },
      handleUpload() {
        const { fileList } = this;

        fileList.forEach(file => {
          this.uploadFile(file);
        });
        console.log(fileList);

        this.uploading = true;

      },
      uploadFile(file){

        const formData = new FormData();
        formData.append('file', file);

        let url = this.url.upload;

        axios.post(url,formData).then((response)=>{
          console.log(response);
          if(response.data.status === 'SUCCESS'){
            this.fileList = [];
            this.uploading = false;
            this.$message.success('上传成功');
          }else{
            this.$message.error('上传失败');
          }
        }).catch((err)=>{
          console.log(err);
          this.uploading = false;
          this.$message.error('上传失败');
        }).finally(()=>{

        });
      }
    },
  }
</script>

<style scoped>

</style>
