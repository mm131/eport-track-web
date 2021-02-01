<template>
  <a-modal
    :title="title"
    :width="400"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <a-spin :spinning="loading">
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
          label="">
          <a-input placeholder="请输入验证码" v-model="code" v-decorator="['code', validatorRules.code]"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { filterObj } from '@/utils/util'
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { getAction, postAction } from '@/api/manage'
  import JDictSelectTag from '@/components/dict/JDictSelectTag'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'


  export default {
    name: 'ManifestNanjingInfoStatusModal',
    mixins: [JeecgListMixin],
    components: { JDictSelectTag },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        title: '获取运抵报告状态',
        visible: false,
        captchaImg: null,
        cid: null,
        jid: null,
        code: '',
        /* table加载状态 */
        loading: false,
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
          list: '/manifestNanjing/getManifest'
        }
      }
    },
    methods: {
      show() {
        this.visible = true
      },
      close() {
        this.form.setFieldsValue({ code: '' })
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        this.loading = true
        getAction(`/manifestNanjing/getManifest`, { jid: this.jid, cid: this.cid, code: this.code }).then(res => {
          if (res.success) {
            this.close()
            this.loading = false
            this.$message.success(`已完成`)
          }
        })
      },
      handleCancel() {
        this.close()
      },
      getCaptchaImg() {
        getAction(`/track/group/captcha`, {}).then(res => {
          if (res.success) {
            this.captchaImg = res.result.img
            this.jid = res.result.jid
            this.cid = res.result.cid
          }
        })
      }
    }
  }
</script>

<style lang="less" scoped>

</style>