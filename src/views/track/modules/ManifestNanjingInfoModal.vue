<template>
  <a-card :bordered="false">
    <a-modal
      :title="title"
      :width="1200"
      :visible="visible"
      :maskClosable="false"
      :confirmLoading="confirmLoading"
      @ok="handleOk"
      @cancel="handleCancel"
      cancelText="关闭">

      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="24">
            <a-col :md="6" :sm="6">
              <a-form-item label="验证码">
                <a-input placeholder="请输入验证码" v-model="code" v-decorator="['code', validatorRules.code]"></a-input>
              </a-form-item>
            </a-col>
            <a-col :md="3" :sm="6">
              <img style="margin-left: 20px;" :src="'data:image/png;base64,'+captchaImg" @click="getCaptchaImg()"/>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :loading="loading"
        @change="handleTableChange">
      </a-table>
    </a-modal>
    <manifest-nanjing-info-arrive-modal ref="ManifestNanjingInfoArriveModal"></manifest-nanjing-info-arrive-modal>
  </a-card>
</template>
<script>
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { getAction, postAction } from '@/api/manage'
  import JDictSelectTag from '@/components/dict/JDictSelectTag'
  import ManifestNanjingInfoArriveModal from './ManifestNanjingInfoArriveModal.vue'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { filterObj } from '@/utils/util'

  export default {
    name: 'ManifestNanjingInfoModal',
    mixins: [JeecgListMixin],
    components: {
      JDictSelectTag,
      ManifestNanjingInfoArriveModal
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        title: '查询运抵报告状态',
        visible: false,
        captchaImg: null,
        uid: null,
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
        },
        visible: false,
        confirmLoading: false,
        columns: [
          {
            title: '#',
            align: 'center',
            dataIndex: 'rowIndex',
            customRender: function(text, r, index) {
              return (text !== '合计') ? (parseInt(index) + 1) : text
            }
          },
          {
            title: '运输工具编号',
            align: 'center',
            dataIndex: 'shipId'
          },
          {
            title: '运输工具名称',
            align: 'center',
            dataIndex: 'shipName'
          },
          {
            title: '航次/班 ',
            align: 'center',
            dataIndex: 'voyageNo'
          },
          {
            title: '提运单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'contaId'
          }, {
            title: '运抵报告状态',
            align: 'center',
            dataIndex: 'njArrivalReceipt'
          }
        ],
        url: {
          list: '/manifestNanjing/getManifestConta'
        }
      }
    },
    methods: {
      show(res) {
        this.visible = true
        this.uid = res.id
      },
      close() {
        this.form.setFieldsValue({ code: '' })
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        this.queryParam.id = this.uid
        this.queryParam.jid = this.jid
        this.queryParam.cid = this.cid
        this.queryParam.code = this.code
        this.loadData()
      },
      handleCancel() {
        this.close()
        this.dataSource = []
      },
      getCaptchaImg() {
        getAction(`/track/group/captcha`, {}).then(res => {
          if (res.success) {
            this.captchaImg = res.result.img
            this.jid = res.result.jid
            this.cid = res.result.cid
          }
        })
      },
      loadData(arg) {
        if (!this.url.list) {
          this.$message.error('请设置url.list属性!')
          return
        }
        if (arg === 1) {
          this.ipagination.current = 1
        }
        var params = this.getQueryParams()
        console.log('查询参数hahaha：{}', params)
        this.loading = true
        getAction(this.url.list, params).then((res) => {
          console.log('返回结果:{}', res)
          if (res.success) {
            this.dataSource = res.result.records
            this.ipagination.total = res.result.total
          }
          if (res.code === 510) {
            this.$message.warning(res.message)
          }
          this.loading = false
        })
      },
      getQueryParams() {
        let sqp = {}
        if (this.superQueryParams) {
          sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        }
        var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
        param.field = this.getQueryField()
        param.pageNo = this.ipagination.current
        param.pageSize = this.ipagination.pageSize
        return filterObj(param)
      }
    }
  }
</script>

<style scoped>

</style>