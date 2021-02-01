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
  </a-card>
</template>

<script>
  import { filterObj } from '@/utils/util'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { getAction } from '@/api/manage'


  export default {
    name: 'ManifestNanjingInfoArriveModal.vue',
    mixins: [JeecgListMixin],
    components: {
      JInput
    },
    data() {
      return {
        title: '查询运抵报告状态',
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
      show() {
        this.visible = true
      },
      handleOk() {
        this.visible = false
      },
      handleCancel() {
        this.visible = false
      },
      load(jid, cid, code, id) {
        this.queryParam.id = id
        this.queryParam.jid = jid
        this.queryParam.cid = cid
        this.queryParam.code = code
        this.loadData()
      },
      loadData(arg) {
        if (!this.url.list) {
          this.$message.error('请设置url.list属性!')
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1
        }
        var params = this.getQueryParams()//查询条件
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
        //获取查询条件
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