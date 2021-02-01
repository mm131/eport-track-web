<template>
  <a-card :bordered="false">
    <a-modal
      :title="title"
      :width="600"
      :visible="visible"
      @ok="handleOk"
      @cancel="handleNotOk"
      cancelText="关闭">

      <a-spin :spinning="confirmLoading">
        <a-form :form="form">

          <a-form-item label="驳船船期" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-date-picker
              v-model="queryParam.shipPeriod"
              show-time
              format="YYYY-MM-DD HH:mm:ss"
              placeholder="请选择船期" style="width:320px"></a-date-picker>
          </a-form-item>


          <a-form-item>
            <a-upload name="file"  :showUploadList="fileRecordShow" :multiple="true"
                      :headers="tokenHeader" :action="uploadUrl"  :before-upload="beforeUpload"  @change="handleImport" style="margin-left: 100px">
              <a-button type="primary" icon="import">上传报文</a-button>
            </a-upload>
          </a-form-item>

        </a-form>
      </a-spin>
    </a-modal>
    <!-- table区域-end -->
  </a-card>
</template>

<script>

  import moment from 'moment'
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import ACol from 'ant-design-vue/es/grid/Col'
  Vue.prototype.$moment = moment

  export default {
    name: 'ManifestShanghaiFaUploadModal',
    components: { ACol },
    data() {
      return {
        title() {
          return '上传报文'
        },
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        visible: false,
        fileRecordShow:false,
        queryParam:{
          shipNameVoyage:null,
          sendMethod:null,
          sendTime: null,
          timeInterval:null,
          periodType:1
        },
        confirmLoading: false,
        form: this.$form.createForm(this),
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 14 }
        },
        importUrl: 'fa/importFA'
      }
    },

    computed: {
      uploadUrl: function() {
        var shipPeriod=this.$moment(this.queryParam.shipPeriod).format('YYYY-MM-DD HH:mm:ss');
        return `${window._CONFIG['domianURL']}/${this.importUrl}?shipPeriod=${shipPeriod}`;
      }
    },


    methods: {
      showUploadPage() {

        this.queryParam={
          shipPeriod:null,
        },
          this.fileRecordShow=false;
        this.visible = true
      },

      moment,

      handleNotOk() {
        this.visible = false;
      },
      handleOk() {
        this.visible = false;
        this.$emit('ok');
      },

      beforeUpload(file){
        if(this.queryParam.shipPeriod==null){
          this.$message.error('请选择驳船船期');
          return false;
        }
        this.fileRecordShow=true;
      },

      handleImport(info) {
        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (info.file.status === 'done') {
          if (info.file.response.success) {
            if (info.file.response.code === 200) {
              this.$message.success(info.file.response.message || `${info.file.name} 文件上传成功`)
            }else{
              this.$message.error('上传失败')
            }
          } else {
            this.$message.error(`${info.file.response.message}`)
          }
        } else if (info.file.status === 'error') {
          this.$message.error(`文件上传失败: ${info.file.msg} `)
        }
      },
    }
  }
</script>

<style scoped>

</style>