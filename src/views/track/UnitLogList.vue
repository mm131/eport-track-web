<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchUnitLogs">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="6">
            <a-form-item label="验证码">
              <a-input placeholder="输入验证码" v-model="queryParam.code"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="3" :sm="6">
            <img style="margin-left: 20px;" :src="'data:image/png;base64,'+captchaImg" @click="getCaptchaImg()"/>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchUnitLogs" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-button type="primary" @click="sendBatch" icon="global" style="margin-left: 8px">批量补发</a-button>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style="border-top: 5px">
      <p>{{this.getLabelDisplay}}</p>
    </div>

    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="boxNo"
        :columns="columns"
        :dataSource="dataSource"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        :scroll="{ x: 'max-content' }"
        :pagination="ipagination"
        @change="handleTableChange">
        <span slot="action" slot-scope="text, record">
      <a href="javascript:;" @click="send(record)">补发</a>
    </span>
      </a-table>
    </div>
    <!-- table区域-end -->
    <send-result-modal ref="sendResultModal"/>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { sendAction, getAction, postAction } from '@/api/manage'
  import SendResultModal from './modules/SendResultModal'

  export default {
    name: 'UnitLogList',
    mixins: [JeecgListMixin],
    components: {
      JInput,
      SendResultModal
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        description: '货箱查询',
        queryParam: {},
        captchaImg: null,
        cid: null,
        jid: null,
        code: '',
        timer: null,
        columns: [
          {
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'boxNo'
          },
          {
            title: '箱型',
            align: 'center',
            dataIndex: 'boxType'
          },
          {
            title: '出门时间',
            align: 'center',
            dataIndex: 'outTime'
          },
          {
            title: '进门时间',
            align: 'center',
            dataIndex: 'inTime'
          },
          {
            title: '装船时间',
            align: 'center',
            dataIndex: 'loadTime'
          },
          {
            title: '南京运抵发送时间',
            align: 'center',
            dataIndex: 'njArrivalSendTime'
          },
          {
            title: '南京运抵回执时间',
            align: 'center',
            dataIndex: 'njArrivalReceiptTime'
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            width: 100
          }
        ],
        ipagination: {
          current: 1,
          pageSize: 10000,
          pageSizeOptions: ['10000'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        /* 排序参数 */
        isorter: {
          column: '',
          order: ''
        },
        url: {
          list: '/unitLog/list',
          sendBatch: '/track/group/sendBatch'
        }
      }
    },
    mounted() {
      clearInterval(this.timer)
      this.getCaptchaImg()
    },
    beforeDestroy() {//页面关闭前关闭定时器
      clearInterval(this.timer)
    },
    computed: {
      getLabelDisplay: function() {
        let count = this.ipagination.total ? this.ipagination.total : 0
        return `查询到 ${count} 个货箱`
        // let str = `查询到 ${this.ipagination.total} 个货箱`
        // if (this.boxCount && parseInt(this.boxCount) > this.ipagination.total) {
        //   str += `，仍缺 ${parseInt(this.boxCount) - this.ipagination.total} 个货箱的信息`
        // }
        // return str
      }
    },
    methods: {
      searchUnitLogs() {
        this.getCaptchaImg()
        if (!this.queryParam.billNo || !this.queryParam.code) {
          this.$message.warning(`提单号 和 验证码为必填`)
          return
        }
        this.searchQuery()
        this.queryParam.code = ''
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
      send(record) {
        postAction(`/track/group/send`, {
          billNo: record.billNo,
          boxNo: record.boxNo,
          shipCode: record.shipCode,
          shipNameEn: record.shipNameEn,
          shipNameCn: record.shipNameCn,
          exVoyage: record.exVoyage,
          shipVisitNo: record.shipVisitNo,
          serviceCode: record.serviceCode,
          cookie: record.cookieCode[0].value
        }).then(res => {
          console.log(res)
          if (res.success) {
            this.viewInfo(res)
          }
        })
      },
      viewInfo(record) {
        this.$refs.sendResultModal.loadSendResult(record)
        this.$refs.sendResultModal.title = '补发结果'
        this.$refs.sendResultModal.show()
      },
      sendBatch(record) {
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
          var aaa = ''
          for (var k = 0; k < this.selectedRowKeys.length; k++) {
            aaa += '{' + this.selectionRows[k].billNo + ',' + this.selectionRows[k].boxNo + ',' + this.selectionRows[k].shipCode + ',' + this.selectionRows[k].shipNameEn + ',' + this.selectionRows[k].shipNameCn + ',' + this.selectionRows[k].exVoyage + ',' + this.selectionRows[k].shipVisitNo + ',' + this.selectionRows[k].serviceCode + ',' + this.selectionRows[k].cookieCode[0].value + '}' + '&'
          }
          var that = this
          this.$confirm({
            title: '确认补发',
            content: '是否批量补发选中数据?',
            style: 'margin-top: 10%',
            onOk: function() {
              that.loading = true
              sendAction(that.url.sendBatch, { record: aaa }).then((res) => {
                if (res.success) {
                  that.batchInfo(res)
                }
              }).finally(() => {
                that.loading = false
              })
            }
          })
        }
      },
      batchInfo(record) {
        this.$refs.sendResultModal.loadBatchResult(record)
        this.$refs.sendResultModal.title = '补发结果'
        this.$refs.sendResultModal.show()
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>