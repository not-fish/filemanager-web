<template>
    <div>
      <a-form-model
        :rules="rules"
        ref="ruleForm"
        :model="form">

        <a-row :gutter="24">
          <a-col :md="6" :sm="8" :offset="3">
            <a-form-model-item label="原文件名" prop="oldFileName">
              <a-auto-complete
                v-model="form.oldFileName"
                :data-source="OldFileNameDataSource"
                placeholder="请输入原文件名"
                style="width: 300px"
                @select="onSelectByOldFileName"
                @search="onSearchByOldFileName"
                @change="onChangeByOldFileName"
              />
            </a-form-model-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <a-form-model-item label="文件名" prop="newFileName">
              <a-auto-complete
                v-model="form.newFileName"
                :data-source="NewFileNameDataSource"
                style="width: 300px"
                placeholder="请输入上传后的文件名"
                @select="onSelectByNewFileName"
                @search="onSearchByNewFileName"
                @change="onChangeByNewFileName"
              />
            </a-form-model-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <a-form-model-item label="类型">
              <a-cascader :options="options" @change="cascaderOnChange" />
            </a-form-model-item>
          </a-col>
          <!--只能选择一个月内的数据-->
          <a-col :md="12" :sm="16" :offset="3">
            <a-form-item label="上传日期">
              <a-space size="middle">
                <a-date-picker
                  :showTime="true"
                  format="YYYY-MM-DD"
                  placeholder="请选择开始时间"
                  v-model="form.startTime"
                  :disabled-date="disabledStartDate"
                  @openChange="handleStartOpenChange">
                  <a-icon slot="suffixIcon" type="smile" />
                </a-date-picker>
                <a-icon type="arrow-right" />
                <a-date-picker
                  :showTime="true"
                  format="YYYY-MM-DD"
                  placeholder="请选择结束时间"
                  v-model="form.endTime"
                  :disabled-date="disabledEndDate"
                  :open="endOpen"
                  @openChange="handleEndOpenChange">
                  <a-icon slot="suffixIcon" type="smile" />
                </a-date-picker>
              </a-space>
            </a-form-item>
          </a-col>

          <a-col :md="24" :sm="24" :offset="16">
            <a-form-model-item :wrapper-col="{ span: 14, offset: 4 }">
              <a-button type="primary" @click="mySearch">
                查询
              </a-button>
              <a-button style="margin-left: 10px;" @click="resetForm">
                重置
              </a-button>
            </a-form-model-item>
          </a-col>

        </a-row>
      </a-form-model>

      <a-row>
        <a-col>
          <a-table
            ref="table"
            size="middle"
            :loading="loading"
            :columns="columns"
            :data-source="dataSource"
            :pagination="ipagination"
            rowKey="id"
            :scroll="{ x: 2000}"
            bordered
            @change="handleTableChange">

            <template slot="action" slot-scope="text">
              <a :href="fileUrl" @click="downloadFile(text)">下载</a>
            </template>

          </a-table>
        </a-col>
      </a-row>

    </div>
</template>

<script>

  import axios from 'axios'
  import moment from 'moment'

  export default {
    name: "FileList",
    mounted() {
      this.list();
    },
    data(){
      return{
        fileUrl:null,
        dataSource:[],
        OldFileNameDataSource:[],
        NewFileNameDataSource:[],
        /* 分页参数 */
        ipagination: {
          current: 1,
          pageSize: 30,
          pageSizeOptions: ['5','10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        /* table加载状态 */
        loading:false,
        /* 排序参数 */
        isorter:{
          //column: 'createTime',
          order: 'desc',
        },
        form: {
            oldFileName:null,
            newFileName:null,
            type:null,
            startTime: moment(),
            endTime: moment(),
        },
        endOpen: false,
        //注意事项：<a-form-model>中的model要和<a-input>中的model绑定的前缀一样，例如这里的是form
        rules: {
          oldFileName: [{ required: false, message: '请输入原文件名'},],
          newFileName: [{ required: false, message: '请输入文件名'}],
        },
        columns: [
          {
            title:'ID',
            align:"center",
            width:60,
            dataIndex: 'id'
          },
          {
            title:'原文件名',
            align:"center",
            ellipsis:true,
            dataIndex: 'oldFileName'
          },
          {
            title:'文件名',
            align:"center",
            ellipsis:true,
            dataIndex: 'newFileName'
          }
          ,
          {
            title:'后缀名',
            align:"center",
            width:100,
            dataIndex: 'ext'
          },
          {
            title:'路径',
            align:"center",
            dataIndex: 'path'
          },
          {
            title:'大小（字节）',
            align:"center",
            dataIndex: 'size'
          },
          {
            title:'类型',
            align:"center",
            width:110,
            dataIndex: 'type'
          },
          {
            title:'是否为图片',
            align:"center",
            width:110,
            dataIndex: 'wonImg',
            customRender:function(wonImg){
              return wonImg === true?'是':'否';
            }
          },
          {
            title:'下载次数',
            align:"center",
            width:110,
            dataIndex: 'downloadCounts'
          },
          {
            title:'上传日期',
            align:"center",
            ellipsis:true,
            dataIndex: 'uploadTime',
            width:200,
            customRender:function (uploadTime) {
              return moment(uploadTime).format("yyyy-MM-DD HH:mm:ss")
            }
          },
          {
            title:'操作',
            align:'center',
            width:100,
            fixed:'right',
            scopedSlots: { customRender: 'action' },
          }
        ],
        options: [
          {
            value: '',
            label: '',
          },
          {
            value: 'image',
            label: 'image',
            children: [
              {
                value: 'png',
                label: 'png',
              },
              {
                value: 'jpeg',
                label: 'jpeg',
              },
            ],
          },
          {
            value: 'text',
            label: 'text',
            children: [
              {
                value: 'javascript',
                label: 'javascript',
              },
            ],
          },
        ],
        url:{
          list: "http://localhost:9090/filemanager/file/list",
          download: "http://localhost:9090/filemanager/file/download",
          query:"http://localhost:9090/filemanager/file/query",
          findNewFileName:"http://localhost:9090/filemanager/file/findNewFileName",
          findOldFileName:"http://localhost:9090/filemanager/file/findOldFileName",
        }
      }
    },
    methods:{
      //触发点击分页，如无这个则没办法点击非当前页
      handleTableChange(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field;
          this.isorter.order = "ascend" === sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
      },
      list(){

        let url = this.url.list;

        axios.get(url).then((response)=>{
          console.log(response);
          this.dataSource = response.data;
        }).catch((err)=>{
          console.log(err);
          this.$message.error('获取文件列表失败');
        }).finally(()=>{

        });
      },
      downloadFile(target){
        console.log('下载');
        console.log(target);

        let url = this.url.download;
        url = url + '?id='+target.id;

        this.fileUrl = url;

      },
      mySearch() {
        this.$refs.ruleForm.validate(valid => {
          if (valid) {
            console.log(this.form);

            let url = this.url.query;

            let data = new FormData();

            if(this.form.oldFileName !== null && this.form.oldFileName !== ""){
              data.append('oldFileName',this.form.oldFileName);
            }
            if(this.form.newFileName !== null && this.form.newFileName !== ""){
              data.append('newFileName',this.form.newFileName);
            }
            if(this.form.type !== null && this.form.type !== ""){
              data.append('type',this.form.type);
            }

            if(this.form.startTime !== null){
              data.append('startTime',this.form.startTime.toString());
            }

            if(this.form.endTime !== null){
              data.append('endTime',this.form.endTime.toString());
            }

            axios.post(url,data).then((response)=>{
              console.log(response);
              this.dataSource = response.data;
            }).catch((err)=>{
              console.log(err);
            }).finally(()=>{

            });

            this.$message.success('成功');
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      resetForm() {
        this.$refs.ruleForm.resetFields();
        this.form.type = null;
        this.onSubmit();
      },
      cascaderOnChange(value){
        console.log(value);
        let options = '';
        value.forEach((e)=>{
          options+=e+'/';
        });

        options = options.substring(0,options.length-1);
        console.log(options);

        this.form.type = options;
      },
      disabledStartDate(startTime) {
        const endTime = this.form.endTime;
        if (!startTime || !endTime) {
          return false;
        }
        const days = endTime.diff(startTime,'days');

        return days >= 31 || days <= (-1);
      },
      disabledEndDate(endTime) {
        const startTime = this.form.startTime;
        if (!endTime || !startTime) {
          return false;
        }
        const days = endTime.diff(startTime,'days');
        return days >= 31 || days <= (-1);
      },
      handleStartOpenChange(open) {
        if (!open) {
          this.endOpen = true;
        }
      },
      handleEndOpenChange(open) {
        this.endOpen = open;
      },
      searchNewFileName(fileName){
        let url = this.url.findNewFileName;
        let data = new FormData();

        if(fileName !== null && fileName !== ""){
          data.append('newFileName',fileName);
        }

        axios.post(url,data).then((response)=>{
          console.log(response);
          this.NewFileNameDataSource = response.data;
        }).catch((err)=>{
          console.log(err);
        }).finally(()=>{

        });
      },

      searchOldFileName(fileName){
        let url = this.url.findOldFileName;
        let data = new FormData();

        if(fileName !== null && fileName !== ""){
          data.append('oldFileName',fileName);
        }

        axios.post(url,data).then((response)=>{
          console.log(response);
          this.OldFileNameDataSource = response.data;
        }).catch((err)=>{
          console.log(err);
        }).finally(()=>{

        });
      },

      onSearchByNewFileName(searchText) {
        console.log('--searching--');
        this.NewFileNameDataSource = [];
        this.searchNewFileName(searchText);
      },
      onSelectByNewFileName(value) {
        console.log('onSelect', value);
      },
      onChangeByNewFileName(value) {
        console.log('onChange', value);
      },

      onSearchByOldFileName(searchText) {
        console.log('--searching--');
        this.OldFileNameDataSource = [];
        this.searchOldFileName(searchText);
      },
      onSelectByOldFileName(value) {
        console.log('onSelect', value);
      },
      onChangeByOldFileName(value) {
        console.log('onChange', value);
      },

    }
  }
</script>

<style scoped>

</style>
