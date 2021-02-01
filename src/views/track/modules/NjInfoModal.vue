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

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="输入验证码">
          <img style="margin-left: 20px;" :src="'data:image/png;base64,'+captchaImg" @click="getCaptchaImg()" />
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
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { getAction, postAction } from '@/api/manage'
  import JDictSelectTag from '@/components/dict/JDictSelectTag'

  export default {
    name: 'NjInfoModal',
    components: { JDictSelectTag },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        title: '补充南京港信息',
        visible: false,
        captchaImg: null,
        cid: null,
        jid: null,
        code: '',
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
        }
      }
    },
    methods: {
      show() {
        this.visible = true
      },
      close() {
        this.form.setFieldsValue({code:''})
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        postAction(`/track/group/njArrivalSend`, {jid: this.jid, cid: this.cid, code: this.code}).then(res=>{
          if(res.success){
            this.$message.success(`正在补充南京港信息，请稍后查询`);
            this.close();
          }
        });
      },
      handleCancel() {
        this.close()
      },
      getCaptchaImg(){
        getAction(`/track/group/captcha`, {}).then(res=>{
          if(res.success){
            this.captchaImg = res.result.img;
            this.jid = res.result.jid;
            this.cid = res.result.cid;
          }
        })
      }
    }
  }
</script>

<style lang="less" scoped>

</style>