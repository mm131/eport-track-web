<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="4" :sm="4">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.sendBillno"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="4" :sm="4">
            <a-form-item label="箱号">
              <a-input placeholder="输入箱号查询" v-model="queryParam.sendBoxid"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="3" :sm="3">
            <a-form-item label="状态">
              <a-select v-model="queryParam.sendStatus" placeholder="全部">
                <a-select-option value=''>全部</a-select-option>
                <a-select-option value=1>成功</a-select-option>
                <a-select-option value=0>失败</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :md="5" :sm="12">
            <a-form-item label="发送时间">
              <a-date-picker
                v-model="startValue"
                :disabled-date="disabledStartDate"
                show-time
                format="YYYY-MM-DD HH:mm:ss"
                placeholder="开始时间"
                @openChange="handleStartOpenChange"
              />
            </a-form-item>
          </a-col>

          <a-col :md="5" :sm="12">
            <a-form-item label="">
              <a-date-picker
                v-model="endValue"
                :disabled-date="disabledEndDate"
                show-time
                format="YYYY-MM-DD HH:mm:ss"
                placeholder="结束时间"
                :open="endOpen"
                @openChange="handleEndOpenChange"
              />
            </a-form-item>
          </a-col>

          <a-col :md="1" :sm="6">
<!--            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">-->
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
<!--            </span>-->
          </a-col>

        </a-row>
      </a-form>
    </div>

    <div>
      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :scroll="{ x: 'max-content' }"
        @change="handleTableChange">

        <span slot="sendStatusType" slot-scope="text, record">
          <span v-if="record.sendStatus===0">失败</span>
          <span v-else-if="record.sendStatus===1">成功</span>
          <span v-else>
            {{record.sendStatus}}
          </span>
        </span>
        <span slot="sendBillnoSlot" slot-scope="text, record">
          <span v-if="record.sendBillno===null">无</span>
          <span v-else-if="record.sendBillno===''">无</span>
          <span v-else>
            {{record.sendBillno}}
          </span>
        </span>
        <span slot="sendBoxidSlot" slot-scope="text, record">
          <span v-if="record.sendBoxid===null">无</span>
          <span v-else-if="record.sendBoxid===''">无</span>
          <span v-else>
            {{record.sendBoxid}}
          </span>
        </span>
      </a-table>
    </div>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import moment from 'moment'

  Vue.prototype.$moment = moment

  export default {
    name: 'YunLspHistoryList',
    mixins: [JeecgListMixin],
    components: {},
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        queryParam: {},
        startValue: null,
        endValue: null,
        endOpen: false,
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
            title: '提单号',
            align: 'center',
            dataIndex: 'sendBillno',
            //scopedSlots: { customRender: 'sendBillnoSlot' }
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'sendBoxid',
            //scopedSlots: { customRender: 'sendBoxidSlot' }
          },
          {
            title: '船名',
            align: 'center',
            dataIndex: 'sendVessel',
          },
          {
            title: '航次',
            align: 'center',
            dataIndex: 'sendVoyNo',
          },
          {
            title: '进出口标识',
            align: 'center',
            dataIndex: 'sendOrderType',
          },
          {
            title: '发送人 ',
            align: 'center',
            dataIndex: 'sendByUsername'
          },
          {
            title: '发送时间',
            align: 'center',
            dataIndex: 'sendTime'
          },
          {
            title: '发送状态',
            align: 'center',
            dataIndex: 'sendStatus',
            scopedSlots: { customRender: 'sendStatusType' }
          },
          {
            title: '失败原因',
            align: 'center',
            dataIndex: 'failureMessage'
          }
        ],
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['20'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: 'yunLspHistory/getYunLspHistory'
        }
      }
    },
    methods: {

      disabledStartDate(startValue) {
        const endValue = this.endValue
        if (!startValue || !endValue) {
          return false
        }
        return startValue.valueOf() > endValue.valueOf()
      },
      disabledEndDate(endValue) {
        const startValue = this.startValue
        if (!endValue || !startValue) {
          return false
        }
        return startValue.valueOf() >= endValue.valueOf()
      },
      handleStartOpenChange(open) {
        if (!open) {
          this.endOpen = true
        }
        this.queryParam.startDate = this.$moment(this.startValue).format('YYYY-MM-DD HH:mm:ss')
      },
      handleEndOpenChange(open) {
        this.endOpen = open
        this.queryParam.endDate = this.$moment(this.endValue).format('YYYY-MM-DD HH:mm:ss')
      },

      searchQuery() {
        if (this.startValue != null && this.endValue == null) {
          this.$message.error('请选择结束时间')
          return
        }
        if (this.startValue == null && this.endValue != null) {
          this.$message.error('请选择开始时间')
          return
        }
        if (this.startValue == null && this.endValue == null) {
          this.queryParam.startDate = null
          this.queryParam.endDate = null
        }
        this.loadData(this.queryParam)
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>