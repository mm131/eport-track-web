<template>

  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    centered
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">


    <div style="margin-top: 10px;">
      <a-form layout="inline" >
        <a-form-item label="选择查看人">
          <a-select
            mode="multiple"
            style="width: 300px;"
            @change="handleChange">
            <a-select-option v-for="(item,index) in userList" :value="item.id"  :key="index">
              {{ item.realname }}
            </a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </div>


    <div style="margin-top: 30px;">
      <a-upload name="file" :showUploadList="false" :multiple="false"
                :headers="tokenHeader" :action="importUrl" @change="handleImport" style="margin-left: 8px;">
        <a-button type="primary" icon="import">上传</a-button>
      </a-upload>
    </div>

  </a-modal>

</template>

<script>

  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { getAction ,postAction} from '@/api/manage'


  export default {
    name: 'ShipmentFormUploadModal',
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        visible:false,
        userList:[],
        viewPermissions:'',
        id:'',
        title:'上传出运表',
        url: {
          importUrl: 'shipmentForm/importShipmentForm'
        }

      }
    },

    computed: {
      importUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}`
      }
    },

    methods: {
      loadUploadPage() {
        getAction(`/sys/user/queryUsersAll`).then(res=>{
          if(res.success){
            this.userList = res.result;
          }
        })
        this.visible=true;
      },


      handleChange(value) {
        this.viewPermissions=`${value}`;
      },

      handleImport(info) {
        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (info.file.status === 'done') {
          if (info.file.response.success) {
            if (info.file.response.code === 200) {
              this.id=info.file.response.result;
              this.$message.success(info.file.response.message || `${info.file.name} 文件上传成功`)
            } else {
              this.$message.error('上传失败')
            }
          } else {
            this.$message.error(`${info.file.response.message}`)
          }
        } else if (info.file.status === 'error') {
          this.$message.error(`文件上传失败: ${info.file.msg} `)
        }
      },

      close () {
        this.$emit('ok');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      },
      handleOk(){
        postAction(`/shipmentForm/insertViewPermissions`,{viewPermissions:this.viewPermissions,id:this.id}).then(res=>{
          if(res.success){
            this.$message.success("文件上传成功！")
          }
        })
        this.close()
      }
    }
  }
</script>


<style scoped="scoped">
</style>

