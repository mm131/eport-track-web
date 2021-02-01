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
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="任务名称">
          <a-input placeholder="请输入任务名称" v-model="name" v-decorator="['name', validatorRules.name]"/>
        </a-form-item>

        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="导入文件">
          <a-upload name="file" :disabled="name.length === 0" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
            <a-button type="primary" icon="import">导入</a-button>
          </a-upload>
        </a-form-item>

      </a-form>
    </a-spin>

  </a-modal>
</template>

<script>
  import pick from 'lodash.pick'
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { httpAction } from '@/api/manage'
  import JDictSelectTag from '@/components/dict/JDictSelectTag'

  export default {
    name: 'BillImportModal',
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        title: '导入',
        visible: false,
        name: '',
        model: {},
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
          name: { rules: [{ required: true, message: '请输入任务名称' }] }
        },
        url: {
          importExcelUrl: 'bill/task/importExcel'
        }
      }
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}?name=${encodeURIComponent(this.name)}`
      }
    },
    methods: {
      show() {
        let that = this
        that.visible = true
        that.model = Object.assign({})
        console.log(that.model)
        that.$nextTick(() => {
          that.form.setFieldsValue(pick(this.model, 'name'))
        })
      },
      handleImportExcel(info) {
        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (info.file.status === 'done') {
          if (info.file.response.success) {
            if (info.file.response.code === 200) {
              this.$message.success(info.file.response.message || `${info.file.name} 文件上传成功`)
              this.close()
            } else {
              this.$message.error('上传失败')
            }
          } else {
            this.$message.error(`${info.file.response.message}`)
            this.close()
          }
        } else if (info.file.status === 'error') {
          this.$message.error(`文件上传失败: ${info.file.msg} `)
        }
      },
      close() {
        this.form.setFieldsValue({ name: '' })
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
      },
      handleCancel() {
        this.close()
      }
    }
  }
</script>

<style scoped>

</style>