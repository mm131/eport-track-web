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

          <a-form-item label="驳船船名航次" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-input v-model="queryParam.shipNameVoyage"></a-input>
          </a-form-item>

          <a-form-item label="发送方式" :labelCol="labelCol" :wrapperCol="wrapperCol">
            <a-select v-model="queryParam.sendMethod" placeholder="未发送" @change="changeMethod">
              <a-select-option value=0>手动发送</a-select-option>
              <a-select-option value=1>自动发送</a-select-option>
              <a-select-option value=2>定时发送</a-select-option>
              <a-select-option value=3>计划靠泊</a-select-option>
              <a-select-option value=4>实际靠泊</a-select-option>
            </a-select>
          </a-form-item>

          <a-form-item label="发送时间" :labelCol="labelCol" :wrapperCol="wrapperCol" v-if="isShow1">
            <a-date-picker
              v-model="queryParam.sendTime"
              show-time
              format="YYYY-MM-DD HH:mm:ss"
              placeholder="请选择发送时间" style="width:320px"></a-date-picker>
          </a-form-item>

          <a-form-item label="提前时间" :labelCol="labelCol" :wrapperCol="wrapperCol"  v-if="isShow2">
            <span>
                <a-input  v-model="queryParam.timeInterval"   style="width: 150px;"></a-input>
                <a-radio-group v-model="queryParam.periodType" style="padding-left: 30px;" @change="onChange">
                  <a-radio :value="1">
                    天
                  </a-radio>
                  <a-radio :value="2">
                    小时
                  </a-radio>
                </a-radio-group>
            </span>

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
  import { getAction, postAction } from '@/api/manage'
  import moment from 'moment'
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import ACol from 'ant-design-vue/es/grid/Col'

  export default {
    name: 'ManifestUploadModal',
    components: { ACol },
    data() {
      return {
        title() {
          return '上传报文'
        },
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        visible: false,
        isShow1:false,
        isShow2:false,
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
        importUrl: 'manifest/importManifest'
      }
    },

    computed: {
      uploadUrl: function() {
        // return `${window._CONFIG['domianURL']}/${this.importUrl}?sendMethod=`+this.queryParam.sendMethod+`&sendTime=`+this.queryParam.sendTime+`&timeInterval=`+
        //   this.queryParam.timeInterval+`&periodType=`+this.queryParam.periodType;
        return `${window._CONFIG['domianURL']}/${this.importUrl}?sendMethod=${this.queryParam.sendMethod}&sendTime=${this.queryParam.sendTime}&timeInterval=${this.queryParam.timeInterval}`+
          `&periodType=${this.queryParam.periodType}&shipNameVoyage=${this.queryParam.shipNameVoyage}`;
      }
    },


    methods: {
      showUploadPage() {

        this.queryParam={
            shipNameVoyage:null,
            sendMethod:null,
            sendTime: null,
            timeInterval:null,
            periodType:1
        },
          this.fileRecordShow=false;
        this.visible = true
      },

      moment,

      changeMethod(value){
        if(value==2){
          this.isShow1=true;
          this.isShow2=false;
        }else if(value==3 ||value==4){
          this.isShow1=false;
          this.isShow2=true;
        } else{
          this.isShow1=false;
          this.isShow2=false;
        }
      },


      onChange(e) {
        console.log('radio checked', e.target.value);
      },

      handleNotOk() {
        this.visible = false;
      },
      handleOk() {
        this.visible = false;
        this.$emit('ok');
      },

      beforeUpload(file){
           if(this.queryParam.sendMethod==null){
             this.$message.error('请选择上传方式');
             return false;
           }
           if(this.queryParam.sendMethod==2 && this.queryParam.sendTime==null){
             this.$message.error('请选择上传时间');
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