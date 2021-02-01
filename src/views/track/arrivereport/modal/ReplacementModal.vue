<template>
  <!--弹框-->
  <a-modal
    :title=title
    :width=width
    :visible=visible
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭"
    footer=''>

    <a-spin :spinning="confirmLoading">
      <a-form @submit="handleSubmit" :form="form">

        <a-row class="form-row" :gutter="24">
          <a-col :span="24" pull="2">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="箱号" style="margin-left: -15px">
              <a-input placeholder="请输入箱号" v-decorator="['unitNo', validatorRules.unitNo]"
                       style="width:50%"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="{ xs: 8, sm: 16, md: 24, lg: 32 }">
          <a-col :span="12">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="船代码" style="margin-right: -35px">
              <a-input placeholder="请输入船代码" v-decorator="['shipCode', validatorRules.shipCode ]"/>
            </a-form-item>
          </a-col>

          <a-col :span="12">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="出口航次">
              <a-input placeholder="请输入出口航次" v-decorator="['shipVoyage', validatorRules.shipVoyage ]"/>
            </a-form-item>
          </a-col>
        </a-row>

        <!--ceshi-->
        <a-row>
          <a-col :span="12">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="船代码" style="margin-right: -35px">
              <a-select placeholder="请选择产品类型" v-decorator="['billNo', validatorRules.billNo ]">
                <a-select-option value="">请选择船代码</a-select-option>
                <a-select-option v-for="(item, key) in this.dataBaseList" :key="key" :value="item.id"
                                 @click="huoqu(item)">
                  {{ item.name }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="出口航次">
              <a-input placeholder="请输入出口航次" v-decorator="['deleteMark', validatorRules.deleteMark ]"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-form-item :wrapperCol="{ span: 24 }" style="text-align: center">
          <a-button htmlType="submit" type="primary">发送</a-button>
          <a-button @click="handleRefresh" style="margin-left: 88px">重置</a-button>
        </a-form-item>

      </a-form>
    </a-spin>

  </a-modal>

</template>

<script>
  import pick from 'lodash.pick'
  import JInput from '@/components/jeecg/JInput'

  export default {
    name: 'ReplacementModal',
    components: { JInput },
    data() {
      return {
        title: '补发',
        width: 800,
        visible: false,
        confirmLoading: false,
        form: this.$form.createForm(this),
        model: {},
        dataBaseList: [
          { id: '1', name: 'jack', deleteMark: '3' },
          { id: '2', name: 'tom', deleteMark: '4' },
          { id: '3', name: 'HarryZhang', deleteMark: '5' }
        ],
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },
        validatorRules: {
          unitNo: { rules: [{ required: true, message: '请输入箱号！' }] },
          shipCode: { rules: [{ required: true, message: '请输入船代码!' }] },
          shipVoyage: { rules: [{ required: true, message: '请输入出口航次!' }] },
          billNo: { rules: [{ required: true, message: '请输入船代码!' }] },
          deleteMark: { rules: [{ required: true, message: '请输入出口航次!' }] }
        }
      }
    },
    methods: {
      huoqu(res) {
        let that = this
        that.visible = true
        that.model = Object.assign({}, res)
        console.log(that.model)
        that.$nextTick(() => {
          that.form.setFieldsValue(pick(this.model, 'deleteMark'))
        })
      },
      reSendTaskInfo(record) {
        let that = this
        that.form.resetFields()
        that.visible = true
        that.model = Object.assign({}, record)
        console.log(that.model)
        that.$nextTick(() => {
          that.form.setFieldsValue(pick(this.model, 'unitNo', 'shipCode', 'shipVoyage'))
        })
      },
      show() {
        this.taskInfo = {}
        this.visible = true
      },
      handleOk() {
        this.visible = false
      },
      handleSubmit(e) {
        e.preventDefault()
        this.form.validateFields((err, values) => {
          if (!err) {
            console.log('单条发送参数： ', values)
          }
        })
      },
      handleRefresh() {
        this.form.resetFields()

      }
    }
  }
</script>

<style scoped>

</style>