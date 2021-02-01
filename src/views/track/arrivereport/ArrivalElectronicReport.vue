<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="箱号">
              <a-input placeholder="输入箱号查询" v-model="queryParam.unitNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-button type="primary" @click="sendBatch" icon="global" style="margin-left: 8px">批量补发</a-button>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- table区域-begin -->
    <a-table
      ref="table"
      size="middle"
      bordered
      rowKey="id"
      :columns="columns"
      :dataSource="dataSource"
      :pagination="ipagination"
      :loading="loading"
      :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
      @change="handleTableChange">

    <span slot="action" slot-scope="text, record">
      <a href="javascript:;" @click="resend(record)">补发</a>
    </span>
    </a-table>

    <!--表单区域-->
    <replacement-modal ref="replacementModal"/>

  </a-card>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import ReplacementModal from './modal/ReplacementModal'
  import { sendAction } from '@/api/manage'


  export default {
    name: 'ArrivalElectronicReport',
    mixins: [JeecgListMixin],
    components: { ReplacementModal },
    data() {
      return {
        description: '运抵电子报告',
        isorter: { column: 'create_time' },
        columns: [
          {
            title: '运抵报告状态',
            align: 'center',
            dataIndex: 'arriveReportState'
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'unitNo'
          },
          {
            title: '船舶代码',
            align: 'center',
            dataIndex: 'shipCode'
          },
          {
            title: '船舶航次',
            align: 'center',
            dataIndex: 'shipVoyage'
          },
          {
            title: '发送时间',
            align: 'center',
            dataIndex: 'sendTime'
          },
          {
            title: '回执时间',
            align: 'center',
            dataIndex: 'receiptTime'
          },
          {
            title: '已回执标志',
            align: 'center',
            dataIndex: 'receiptMark'
          },
          {
            title: '删除标志',
            align: 'center',
            dataIndex: 'deleteMark'
          },
          {
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '箱ISO代码',
            align: 'center',
            dataIndex: 'unitIsoCode'
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            width: 100
          }
        ],
        url: {
          list: '/arrive/task/list',
          sendBatch: '/arrive/task/sendBatch'
        }
      }
    },
    methods: {
      resend(record) {
        this.$refs.replacementModal.reSendTaskInfo(record)
        this.$refs.replacementModal.title = '补发'
        this.$refs.replacementModal.show()
      },
      sendBatch: function() {
        if (!this.url.sendBatch) {
          this.$message.error('请设置url.sendBatch属性!')
          return
        }
        if (this.selectedRowKeys.length <= 0) {
          this.$confirm({
            title: '请选择一条或多条记录',
            style: 'margin-top: 10%'
          })
          return
        } else {
          var ids = ''
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            ids += this.selectedRowKeys[a] + ','
          }
          console.log('选中的数据', ids)
          var that = this
          this.$confirm({
            title: '确认补发',
            content: '是否批量补发选中数据?',
            style: 'margin-top: 10%',
            onOk: function() {
              that.loading = true
              sendAction(that.url.sendBatch, { ids: ids }).then((res) => {
                console.log(res)
                 /*if (res.success) {
                   that.$message.success(res.message)
                   that.loadData()
                   that.onClearSelected()
                 } else {
                   that.$message.warning(res.message)
                 }*/
              }).finally(() => {
                that.loading = false
              })
            }
          })
        }
      }
    }
  }
</script>

<style scoped>
  @import '~@assets/less/common.less';
</style>