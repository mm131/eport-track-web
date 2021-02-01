<template>
  <a-modal
    :title="title"
    :width="400"
    :visible="visible"
    :confirmLoading="confirmLoading"
    :footer=footer
    @ok="handleOk"
    @cancel="handleCancel">

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="输入验证码">
          <img style="margin-left: 20px;" :src="'data:image/png;base64,'+captchaImg" @click="getCaptchaImg()"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="验证码">
          <a-input placeholder="请输入验证码" v-model="code" v-decorator="['code', validatorRules.code]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="导入文件">
          <a-upload name="file" :disabled="code.length === 0" :showUploadList="false" :multiple="false"
                    :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
            <a-button type="primary" icon="import">导入</a-button>
          </a-upload>
        </a-form-item>
      </a-form>
    </a-spin>

  </a-modal>
</template>

<script>
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { httpAction } from '@/api/manage'
  import JDictSelectTag from '@/components/dict/JDictSelectTag'
  import { sendAction, getAction, postAction } from '@/api/manage'

  export default {
    name: 'ContainerImportModal',
    components: { JDictSelectTag },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        title: '导入',
        visible: false,
        footer: null,
        name: '',
        code: '',
        model: {},
        queryParam: {},
        captchaImg: null,
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {
          code: { rules: [{ required: true, message: '请输入验证码' }] }
        },
        url: {
          // importExcelUrl: 'shipment/task/importExcel'
          importExcelUrl: 'shipment/task/importExcelAsync'
        }
      }
    },
    computed: {},
    methods: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}?jid=${encodeURIComponent(this.queryParam.jid)}&cid=${encodeURIComponent(this.queryParam.cid)}&orgCode=${encodeURIComponent(this.queryParam.orgCode)}&code=${encodeURIComponent(this.code)}&t=` + Math.random()
      },
      getCaptchaImg() {
        getAction(`/track/group/captcha`, {}).then(res => {
          if (res.success) {
            this.captchaImg = res.result.img
            this.queryParam.jid = res.result.jid
            this.queryParam.cid = res.result.cid
          }
        })
      },
      show() {

        this.visible = true
        this.queryParam.orgCode = Vue.ls.get('Login_Userinfo').orgCode

      },
      cleanCode() {
        this.form.setFieldsValue({ code: '' })
      },
      handleImportExcel(info) {
        this.confirmLoading = true

        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (info.file.status === 'done') {
          if (info.file.response.success) {
            if (info.file.response.code === 200) {
              this.$message.success(info.file.response.message || `${info.file.name} 文件上传成功`)

              this.confirmLoading = false
              this.close()
            } else {
              this.$message.error('上传失败')
            }
          } else {
            this.confirmLoading = false
            this.$message.error(`${info.file.response.message}`)
            this.close()
          }
        } else if (info.file.status === 'error') {
          this.$message.error(`文件上传失败: ${info.file.msg} `)
          this.confirmLoading = false
        }
      },
      close() {
        this.form.setFieldsValue({ code: '' })
        this.$emit('close')
        this.visible = false
      },
      /*handleOk() {

        this.confirmLoading = true
        getAction(`/shipment/task/importExcel`, { jid: this.jid, cid: this.cid, code: this.code }).then(res => {
          /!* if (res.success) {
             this.close()
             this.loading = false
             this.$message.success(`已完成`)
           }*!/
        })
      },*/
      handleCancel() {
        this.close()
      },
      handleOk() {

      }
    }
  }
</script>


<style lang="less" scoped>

</style>