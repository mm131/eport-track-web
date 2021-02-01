<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="名称">
              <a-input placeholder="输入任务名称查询" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style="border-top: 5px">
      <a-button @click="downloadTemplate" type="primary" icon="download">模板</a-button>
      <a-button @click="handleGroupImport()" type="primary" icon="import">导入</a-button>
      <a-button @click="handleNjInfo()" type="primary" icon="reload">南京港</a-button>
    </div>

    <!-- table区域-begin -->
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

        <span slot="action" slot-scope="text, record">
          <a href="javascript:;" @click="viewGroupDetail(record)">详情</a>
        </span>

        <!--提单书/箱数-->
        <span slot="billNoCountAndBoxNoCount" slot-scope="text, record">
            {{record.billCount+ '个/' + record.boxCount + '箱'+'('}}
          <a href="javascript:;" v-bind:style="{ color: parseInt(record.autoHoldDescCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail1(record, 'auto_hold_desc')">
            {{ record.autoHoldDescCount}}
          </a>
          {{'箱)'}}
        </span>


        <!-- 南京海关未放行 -->
        <!--<span slot="njCustomNotRelease" slot-scope="text, record">
          <a href="javascript:;" v-bind:style="{ color: parseInt(record.boxCount) - parseInt(record.njCustomReleaseCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'nj_custom_release_time')">
            {{parseInt(record.boxCount) - parseInt(record.njCustomReleaseCount)}}
          </a>箱
        </span>-->
        <span slot="njCustomNotRelease" slot-scope="text, record">
          <a href="javascript:;" v-bind:style="{ color: parseInt(record.njCustomReleaseCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'nj_custom_release_time')">
                {{record.njCustomReleaseCount}} ({{record.njCustomReleaseTimeCount_check}})
          </a>箱
        </span>


        <!-- 南京港运抵未发送 -->
        <!--<span slot="njNotArrivalSend" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.boxCount) - parseInt(record.njArrivalSendCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'nj_arrival_send_time')">
            {{parseInt(record.boxCount) - parseInt(record.njArrivalSendCount)}}
          </a>箱
        </span>-->
        <span slot="njNotArrivalSend" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.njArrivalSendCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'nj_arrival_send_time')">

            <span v-if="record.njArrivalSendCount===0">{{record.njArrivalSendCount}}</span>
            <span v-else>
               {{parseInt(record.njArrivalSendCount)}}
              <!--({{record.njArrivalSendTime_check}})-->
            </span>

          </a>箱
        </span>

        <!-- 南京港运抵未回执 -->
        <!--<span slot="njNotArrivalReceipt" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.boxCount) - parseInt(record.njArrivalReceiptCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'nj_arrival_receipt_time')">
            {{parseInt(record.boxCount) - parseInt(record.njArrivalReceiptCount)}}
          </a>箱
        </span>-->
        <span slot="njNotArrivalReceipt" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.njArrivalReceiptCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'nj_arrival_receipt_time')">

            <span v-if="record.njArrivalReceiptCount===0">{{record.njArrivalReceiptCount}}</span>
            <span v-else>
               {{parseInt(record.njArrivalReceiptCount)}}
              <!--({{record.njArrivalReceiptTimeCount_check}})-->
            </span>

          </a>箱
        </span>

        <!-- 上海港未进场 -->
        <!--<span slot="shNotArrivePort" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.boxCount) - parseInt(record.shArrivePortCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'arrival_port_time')">
            {{parseInt(record.boxCount) - parseInt(record.shArrivePortCount)}}
          </a>箱
        </span>-->
        <span slot="shNotArrivePort" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.shArrivePortCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'arrival_port_time')">

            <span v-if="record.shArrivePortCount===0">{{record.shArrivePortCount}}</span>
            <span v-else>
              {{ parseInt(record.shArrivePortCount)}}
              <!--({{record.arrivalPortTimeCount_check}})-->
            </span>

          </a>箱
        </span>

        <!-- 上海港海关未放行 -->
        <!--<span slot="shCustomNotRelease" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.boxCount) - parseInt(record.shCustomReleaseCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'sh_custom_release_time')">
            {{parseInt(record.boxCount) - parseInt(record.shCustomReleaseCount)}}
          </a>箱
        </span>-->
        <span slot="shCustomNotRelease" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color:  parseInt(record.shCustomReleaseCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'sh_custom_release_time')">

            <span v-if="record.shCustomReleaseCount===0">{{record.shCustomReleaseCount}}</span>
            <span v-else>
              {{parseInt(record.shCustomReleaseCount)}}
              <!-- ({{record.shCustomReleaseTimeCount_check}})-->
            </span>

          </a>箱
        </span>

        <!-- 上海港码头未放行 -->
        <!-- <span slot="shPortNotRelease" slot-scope="text, record">
           <a href="javascript:;"
              v-bind:style="{ color: parseInt(record.boxCount) - parseInt(record.shPortReleaseCount) > 0 ? 'red' : 'black' }"
              @click="viewGroupDetail(record, 'port_release_time')">
             {{parseInt(record.boxCount) - parseInt(record.shPortReleaseCount)}}
           </a>箱
         </span> -->
        <span slot="shPortNotRelease" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.shPortReleaseCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'port_release_time')">

            <span v-if="record.shPortReleaseCount===0">{{record.shPortReleaseCount}}</span>
            <span v-else>
             {{ parseInt(record.shPortReleaseCount)}}
              <!-- ({{record.portReleaseTimeCount_check}})-->
            </span>

          </a>箱
        </span>

        <!-- 上海港未出场 -->
        <!--<span slot="shNotLeavePort" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.boxCount) - parseInt(record.shLeavePortCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'leave_port_time')">
            {{parseInt(record.boxCount) - parseInt(record.shLeavePortCount)}}
          </a>箱
        </span>-->
        <span slot="shNotLeavePort" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.shLeavePortCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record, 'leave_port_time')">

            <span v-if="record.shLeavePortCount===0">{{record.shLeavePortCount}}</span>
            <span v-else>
             {{ parseInt(record.shLeavePortCount)}}
              <!--  ({{record.leavePortTimeCount_check}})-->
            </span>

          </a>箱
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <group-import-modal ref="groupImportModal" @close="importModalOk" @ok="importModalOk"></group-import-modal>
    <group-detail-modal ref="groupDetailModal" @close="importModalOk" @ok="importModalOk"></group-detail-modal>
    <nj-info-modal ref="njInfoModal" @close="importModalOk" @ok="importModalOk"></nj-info-modal>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import GroupImportModal from './modules/GroupImportModal'
  import GroupDetailModal from './modules/GroupDetailModal'
  import NjInfoModal from './modules/NjInfoModal'

  export default {
    name: 'TaskList',
    mixins: [JeecgListMixin],
    components: {
      JInput,
      GroupImportModal,
      GroupDetailModal,
      NjInfoModal
    },
    data() {
      return {
        description: '货运追踪任务组',
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        queryParam: {},
        timer: null,
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
            title: '日期',
            align: 'center',
            dataIndex: 'createTime'
          },
          {
            title: '名称',
            align: 'center',
            dataIndex: 'name'
          },
          {
            title: '提单数/箱数（锁定）',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'billNoCountAndBoxNoCount' }
          },
          {
            title: '南京港运抵未发送',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'njNotArrivalSend' }
          },
          {
            title: '南京港运抵未回执',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'njNotArrivalReceipt' }
          },
          {
            title: '南京海关未放行（查验中）',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'njCustomNotRelease' }
          },
          // {
          //   title: '上海港未进场',
          //   align: 'center',
          //   // width: 80,
          //   dataIndex: '',
          //   scopedSlots: { customRender: 'shNotArrivePort' }
          // },
          // {
          //   title: '上海港海关未放行',
          //   align: 'center',
          //   // width: 100,
          //   dataIndex: '',
          //   scopedSlots: { customRender: 'shCustomNotRelease' }
          // },
          // {
          //   title: '上海港码头未放行',
          //   align: 'center',
          //   // width: 80,
          //   dataIndex: '',
          //   scopedSlots: { customRender: 'shPortNotRelease' }
          // },
          // {
          //   title: '上海港未出场',
          //   align: 'center',
          //   // width: 80,
          //   dataIndex: '',
          //   scopedSlots: { customRender: 'shNotLeavePort' }
          // },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            fixed: 'right',
            width: 100
          }
        ],
        url: {
          list: '/track/group/list',
          delete: '/track/group/delete',
          deleteBatch: '/track/group/deleteBatch',
          templateUrl: 'track/task/template',
          importExcelUrl: 'track/group/importExcel'
        }
      }
    },
    mounted() {
      clearInterval(this.timer)
      this.setTimer()
    },
    beforeDestroy() {//页面关闭前关闭定时器
      clearInterval(this.timer)
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      setTimer: function() {
        this.timer = setInterval(() => {
          this.searchQuery()
        }, 60000)
      },
      downloadTemplate: function() {
        let url = `${window._CONFIG['domianURL']}/${this.url.templateUrl}`
        window.location.href = url
      },
      handleMenuClick(e) {
        if (e.key == 1) {
          this.batchDel()
        }
      },
      handleGroupImport() {
        this.$refs.groupImportModal.show()
      },
      handleNjInfo() {
        this.$refs.njInfoModal.getCaptchaImg()
        this.$refs.njInfoModal.show()
      },
      importModalOk() {
        this.searchQuery()
      },
      viewGroupDetail(record, filterColumn) {
        this.$refs.groupDetailModal.resetScreenSize()
        this.$refs.groupDetailModal.queryParam = { groupId: record.id, filterColumn: filterColumn }
        this.$refs.groupDetailModal.ipagination.current = 1
        this.$refs.groupDetailModal.loadData()
        this.$refs.groupDetailModal.title = '提单追踪明细'
      },

      viewGroupDetail1(record, filterColumn) {
        this.$refs.groupDetailModal.resetScreenSize()
        this.$refs.groupDetailModal.queryParam = { groupId: record.id, filterAutoHoldDesc: filterColumn }
        this.$refs.groupDetailModal.ipagination.current = 1
        this.$refs.groupDetailModal.loadData()
        this.$refs.groupDetailModal.title = '提单追踪明细'
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>