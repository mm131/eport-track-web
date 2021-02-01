<template>
  <a-card :bordered="false">
    <a-modal
      :title="title"
      :width="400"
      :visible="visible"
      @ok="handleOk"
      @cancel="handleNotOk"
      cancelText="关闭">

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

        </a-form>
      </a-spin>
    </a-modal>
    <!-- table区域-end -->
    <send-result-modal ref="sendResultModal"/>
  </a-card>
</template>

<script>
  import { sendAction, getAction, postAction } from '@/api/manage'
  import SendResultModal from './SendResultModal'


  export default {
    name: 'ManifestNanjingContaCodeModal',
    components: {
      SendResultModal
    },
    data() {
      return {
        title() {
          return '补发'
        },
        visible: false,
        code: '',
        captchaImg: null,
        confirmLoading: false,
        queryParam: {},
        form: this.$form.createForm(this),
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },
        validatorRules: {
          code: { rules: [{ required: true, message: '请输入验证码' }] }
        }
      }
    },
    methods: {
      show(res) {
        this.queryParam.data = res
        this.visible = true
      },
      handleNotOk() {
        this.visible = false
      },
      handleOk() {
        if (!this.code) {
          this.$message.warn('请输入验证码')
        } else {
          this.confirmLoading = true

          postAction(`/track/group/sendOfNanjingYuPei?t=${encodeURIComponent(Math.random())}`, {
            billNo: this.queryParam.data.billNo,
            boxNo: this.queryParam.data.contaId,
            shipCode: '',
            shipNameEn: this.queryParam.data.shipName,
            shipNameCn: '',
            exVoyage: this.queryParam.data.voyageNo,
            shipVisitNo: '',
            serviceCode: '',
            cookie: '',
            jid: this.queryParam.jid,
            cid: this.queryParam.cid,
            code: this.code
          }).then(res => {
            console.log(res)
            if (res.success) {
              this.confirmLoading = false
              this.visible = false
              this.viewInfo(res)
              this.form.setFieldsValue({ code: '' })
            }
          })
        }
      },
      viewInfo(record) {
        this.$refs.sendResultModal.loadSendResult(record)
        this.$refs.sendResultModal.title = '补发结果'
        this.$refs.sendResultModal.show()
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
      cleanCode() {
        this.form.setFieldsValue({ code: '' })
      }
    }
  }
</script>

<style scoped>

</style>