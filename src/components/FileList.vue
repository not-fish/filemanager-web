<template>
    <div>
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
          <a @click="downloadFile(text)">下载</a>
        </template>

      </a-table>
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
        dataSource:[],
        /* 分页参数 */
        ipagination: {
          current: 1,
          pageSize: 5,
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
            title:'上传时间',
            align:"center",
            ellipsis:true,
            dataIndex: 'uploadTime',
            width:200,
            customRender:function (uploadTime) {
              return moment(uploadTime).format("yyyy-MM-DD HH:mm:ss:SSS")
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
        url:{
          list: "http://localhost:9090/filemanager/file/list",
          download: "http://localhost:9090/filemanager/file/download",
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

        axios.get(url).then((response)=>{
          console.log(response);
        }).catch((err)=>{
          console.log(err);
        }).finally(()=>{

        });

      }
    }
  }
</script>

<style scoped>

</style>
