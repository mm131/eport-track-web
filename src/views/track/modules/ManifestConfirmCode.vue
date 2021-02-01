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
            <a-input placeholder="请输入验证码或计算结果" v-model="code"/>
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

  export default {
    name: 'ManifestConfirmCode',
    components: {},
    data() {
      return {
        title() {
          return '验证码'
        },
        visible: false,
        code: '',
        session: '',
        captchaImg: null,
        confirmLoading: false,
        url: '',
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
          code: { rules: [{ required: true, message: '请输入验证码或计算结果' }] }
        },

        idArr:"",
        billNoArr:"",

      }
    },

    methods: {
      showConfirmCode(record,type) {
        this.code = '';

        if(type===1){
            this.idArr=record.id;
            this.billNoArr=record.billNo;
            if (record.receiverCode === '上海华港(13453884-2)') {
              this.url = '/cpsa'
            } else if (record.receiverCode === '上港联代(133747781)') {
              this.url = '/unisco'
            } else {
              this.url = ''
            }
        }

        if(type===2){
            var idString='';
            var billNoString='';
            for(var i=0;i<record.length;i++){
              if(billNoString.length>0){
                idString=idString+','+record[i].id;
                billNoString=billNoString+','+record[i].billNo;
              }else{
                idString=idString+record[i].id;
                billNoString=billNoString+record[i].billNo;
              }
            }
            this.idArr=idString
            this.billNoArr=billNoString;

            if (record[0].receiverCode === '上海华港(13453884-2)') {
              this.url = '/cpsa'
            } else if (record[0].receiverCode === '上港联代(133747781)') {
              this.url = '/unisco'
            } else {
              this.url = ''
            }
        }

        this.visible = true
        this.getCaptchaImg()
      },

      moment,

      handleNotOk() {
        this.visible = false
      },
      handleOk() {
        if (!this.code) {
          this.$message.warn('请输入验证码或计算结果')
        } else {
          this.confirmLoading = true;
          console.log('url:' + this.url)
          postAction(this.url + `/automaticConfirmation?t=${encodeURIComponent(Math.random())}`, {
            id: this.idArr,
            billNbr: this.billNoArr,
            verCode: this.code,
            sid: this.session
          }).then(res => {
            if (res.success) {
              this.confirmLoading = false
              this.visible = false
              if (res.result.result == true) {
                this.$message.success('确认成功')
              } else if (res.result.result == false) {
                this.$message.error('确认失败：' + res.result.error)
              }
            } else {
              this.confirmLoading = false
              this.visible = false
              this.$message.error('确认失败：' + res.message)
            }
          })
        }
      },

      getCaptchaImg() {
        console.log('url:' + this.url)
        postAction(this.url + `/getCaptcha?t=${encodeURIComponent(Math.random())}`, {
        }).then(res => {
          if (res.success) {
            console.log(res.result)
            if (res.result.result == true) {
              this.captchaImg = res.result.img
              this.session = res.result.sid
            } else if (res.result.result == false) {
              this.confirmLoading = false
              this.visible = false
              this.$message.error('确认失败：' + res.result.error)
            }
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