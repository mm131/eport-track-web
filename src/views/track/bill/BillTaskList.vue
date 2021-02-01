<template>
  <a-card :bordered="false">

    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <a-form-item label="状态">
              <a-select v-model="queryParam.status" placeholder="未装船">
                <a-select-option value="2">全部</a-select-option>
                <a-select-option value="0">未装船</a-select-option>
                <a-select-option value="1">已装船</a-select-option>
              </a-select>
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
    <!--<div class="table-operator" style="margin-top: 5px">
      <a-button @click="downloadTemplate" type="primary" icon="download">模板</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
    </div>-->

    <a-table
      ref="table"
      size="middle"
      bordered
      rowKey="billNo"
      :columns="columns"
      :dataSource="dataSource"
      :pagination="ipagination"
      :loading="loading"
      @change="handleTableChange">

      <!--南京港未进场箱数-->
      <span slot="njNotYardApproach" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.inYardTimeCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record,record.inYardTimeCount,'in_yard_time')">
            {{record.inYardTimeCount}}
          </a>
      </span>

      <!--南京港未发运抵箱数-->
      <span slot="njNotArrivalSend" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.njArrivalSendTimeCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record,record.njArrivalSendTimeCount,'nj_arrival_send_time')">
            {{record.njArrivalSendTimeCount}}</a>
      </span>

      <!--南京港运抵未回执箱数-->
      <span slot="njNotArrivalReceipt" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.njArrivalReceiptTimeCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record,record.njArrivalReceiptTimeCount,'nj_arrival_receipt_time')">
            {{record.njArrivalReceiptTimeCount}}</a>
      </span>

      <!--南京港运抵未装船箱数-->
      <span slot="njNotLoad" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.njLoadTimeCount) > 0 ? 'red' : 'black' }"
             @click="viewGroupDetail(record,record.njLoadTimeCount,nj_load_time)">
            {{record.njLoadTimeCount}}</a>
      </span>

      <span slot="action" slot-scope="text, record">
<!--          <a href="javascript:;" @click="viewInfo(record)">货箱查询</a>-->
          <a href="javascript:;" @click="viewGroupDetail(record)">货箱查询</a>
        <!--          <a-divider type="vertical"/>-->
        <!--<a href="javascript:;" @click="send(record)">补发</a>-->
      </span>

    </a-table>

    <bill-task-info-modal ref="billTaskInfoModal"/>
    <group-detail-modal ref="groupDetailModal"/>

  </a-card>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import BillTaskInfoModal from './modal/BillTaskInfoModal'
  import GroupDetailModal from './modal/GroupDetailModal'
  import { postAction } from '@/api/manage'


  export default {
    name: 'BillTaskList',
    mixins: [JeecgListMixin],
    components: {
      BillTaskInfoModal,
      GroupDetailModal
    },
    data() {
      return {
        description: '提单管理页面',
        isorter: { column: 'create_time' },
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
            dataIndex: 'billNo'
          },
          {
            title: '总箱数',
            align: 'center',
            dataIndex: 'boxCount'
          },
          {
            title: '南京港未进场箱数',
            align: 'center',
            dataIndex: 'inYardTimeCount',
            scopedSlots: { customRender: 'njNotYardApproach' }
          },
          {
            title: '南京港未发运抵箱数',
            align: 'center',
            dataIndex: 'njArrivalSendTimeCount',
            scopedSlots: { customRender: 'njNotArrivalSend' }
          },
          {
            title: '南京港运抵未回执箱数',
            align: 'center',
            dataIndex: 'njArrivalReceiptTimeCount',
            scopedSlots: { customRender: 'njNotArrivalReceipt' }
          },
          {
            title: '南京港运抵未装船箱数',
            align: 'center',
            dataIndex: 'njLoadTimeCount',
            scopedSlots: { customRender: 'njNotLoad' }
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            width: 150
          }
        ],
        url: {
          list: '/bill/task/list',
          importExcelUrl: 'bill/task/importExcel'
        }
      }
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      viewInfo(record) {
        this.$refs.billTaskInfoModal.loadBillTaskInfo(record.billNo)
        this.$refs.billTaskInfoModal.title = '提单详情'
        this.$refs.billTaskInfoModal.show()
      },
      viewGroupDetail(record, count, index) {
        this.$refs.groupDetailModal.loadData(record.billNo, count, index)
        this.$refs.groupDetailModal.title = '提单明细'
        this.$refs.groupDetailModal.show()
      },
      send(record) {
        console.log(record.billNo)
        postAction(`/track/group/billSend`, {
          billNo: record.billNo
          /*boxNo: record.boxNo,
          shipCode: record.shipCode,
          shipNameEn: record.shipNameEn,
          shipNameCn: record.shipNameCn,
          exVoyage: record.exVoyage,
          shipVisitNo: record.shipVisitNo,
          serviceCode: record.serviceCode,
          cookie: record.cookieCode[0].value*/
        }).then(res => {
          console.log(res)
        })
      },
      downloadTemplate: function() {
        let url = `${window._CONFIG['domianURL']}/${this.url.templateUrl}`
        window.location.href = url
      },
      handleImportExcel(info) {
        console.log(info)
        // if (info.file.status !== 'uploading') {
        //   console.log(info.file, info.fileList)
        // }
        /*if (info.file.status === 'done') {
          if (info.file.response.success) {
            this.$message.success(`${info.file.name} 文件上传成功`)
            if (info.file.response.code === 201) {
              let { message, result: { msg, fileUrl, fileName } } = info.file.response
              let href = window._CONFIG['domianURL'] + fileUrl
              this.$warning({
                title: message,
                content: (
                  <div>
                    <span>{msg}</span><br/>
                    <span>具体详情请 <a href={href} target="_blank" download={fileName}>点击下载</a> </span>
                  </div>
                )
              })
            } else {
              this.$message.success(info.file.response.message || `${info.file.name} 文件上传成功`)
            }
            this.loadData()
          } else {
            this.$message.error(`${info.file.name} ${info.file.response.message}.`)
          }
        } else if (info.file.status === 'error') {
          this.$message.error(`文件上传失败: ${info.file.msg} `)
        }*/
      }

    }
  }
</script>

<style scoped>
  @import '~@assets/less/common.less'
</style>