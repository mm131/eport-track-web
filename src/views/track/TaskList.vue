<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="箱号">
              <a-input placeholder="输入箱号查询" v-model="queryParam.boxNo"></a-input>
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
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style="border-top: 5px">
      <a-button @click="downloadTemplate" type="primary" icon="download">模板</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay" @click="handleMenuClick">
          <a-menu-item key="1">
            <a-icon type="delete" @click="batchDel"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i>已选择&nbsp;<a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项&nbsp;&nbsp;
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a href="javascript:;" @click="viewInfo(record)">详情</a>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->
    <task-info-modal ref="taskInfoModal" @close="taskInfoModalOk" @ok="taskInfoModalOk"></task-info-modal>
  </a-card>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import TaskInfoModal from './modules/TaskInfoModal'

  export default {
    name: 'TaskList',
    mixins: [JeecgListMixin],
    components: {
      JInput,
      TaskInfoModal
    },
    data() {
      return {
        description: '货运追踪任务',
        queryParam: {},
        columns: [
          {
            title: '箱号',
            align: 'center',
            width: 100,
            dataIndex: 'boxNo'
          },
          {
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo',
            width: 120
          },
          {
            title: '箱主',
            align: 'center',
            width: 100,
            dataIndex: 'boxOwner'
          },
          {
            title: '箱型',
            align: 'center',
            width: 80,
            dataIndex: '',
            customRender(task) {
              return task.boxType + task.boxSize
            }
          },
          {
            title: '南京海关放行时间',
            align: 'center',
            width: 100,
            dataIndex: 'njCustomReleaseTime'
          },
          {
            title: '出口船名',
            align: 'center',
            width: 80,
            dataIndex: 'englishVessel'
          },
          {
            title: '出口航次',
            align: 'center',
            width: 80,
            dataIndex: 'leaveVoyno'
          },
          {
            title: '码头',
            align: 'center',
            width: 80,
            dataIndex: 'dockCode'
          },
          {
            title: '进场时间',
            align: 'center',
            width: 80,
            dataIndex: 'arrivalPortTime'
          },
          {
            title: '运抵报文时间',
            align: 'center',
            width: 80,
            dataIndex: 'arrivalMessageSendTime'
          },
          {
            title: '上海海关放行时间',
            align: 'center',
            width: 80,
            dataIndex: 'shCustomReleaseTime'
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            width: 80
          }
        ],
        url: {
          list: '/track/task/list',
          delete: '/track/task/delete',
          deleteBatch: '/track/task/deleteBatch',
          templateUrl: 'track/task/template',
          importExcelUrl: 'track/task/importExcel'
        }
      }
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      downloadTemplate: function() {
        let url = `${window._CONFIG['domianURL']}/${this.url.templateUrl}`
        window.location.href = url
      },
      handleMenuClick(e) {
        if (e.key == 1) {
          this.batchDel()
        }
      },
      viewInfo(record) {
        // if(record.id && record.boxNo && record.billNo){
        //   this.$router.push({path:'/track/task/info', query: {id: record.id, boxNo: record.boxNo, billNo: record.billNo}})
        // }
        this.$refs.taskInfoModal.loadTaskInfo(record.id)
        this.$refs.taskInfoModal.title = '任务详情'
        this.$refs.taskInfoModal.show()
      },
      taskInfoModalOk() {

      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>