<template>
  <a-modal
    :title="title"
    :width="1300"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">


    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :span="8">
            <a-form-item label="提单号">
              <a-input placeholder="请输入提单号" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item label="箱号">
              <a-input placeholder="请输入箱号" v-model="queryParam.boxNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <j-checkbox style="margin-left: 30px" v-model="jcheckbox.values" :options="jcheckbox.options"
                          @change="getlock(jcheckbox.values)"/>
            </span>
          </a-col>
        </a-row>
      </a-form>

    </div>

    <a-table
      ref="table"
      size="middle"
      bordered
      rowKey="id"
      :columns="columns"
      :dataSource="dataSource"
      :pagination="ipagination"
      :scroll="{ x: 'max-content' }"
      @change="handleTableChange">
      <span slot="action" slot-scope="text, record">
        <a href="javascript:;" @click="viewTask(record)">详情</a>
      </span>
      <div>海关锁</div>

      <span slot="boxNOAutoHoldDesc" slot-scope="text, record">
        <span v-if="record.autoHoldDesc===null">{{record.boxNo}}</span>
        <span v-else><a-icon type="lock" :title="record.autoHoldDesc" style="color: Purple;" v-on:mouseenter="seeTip"
                             @mouseleave="disSeeTip"/>{{record.boxNo}} </span>
      </span>


    </a-table>

    <task-info-modal ref="taskInfoModal" @close="taskInfoModalOk" @ok="taskInfoModalOk"></task-info-modal>
  </a-modal>
</template>

<script>
  import { filterObj } from '@/utils/util'
  import { getGroupDetails } from '@/api/api'
  import TaskInfoModal from './TaskInfoModal'
  import JCheckbox from '@/components/jeecg/JCheckbox'


  export default {
    name: 'GroupDetailModal',
    components: {
      TaskInfoModal,
      JCheckbox
    },
    data() {
      return {
        title: '提单追踪明细',
        queryParam: {},
        seen: false,
        jcheckbox: {
          values: null,
          options: [
            { label: '仅显示锁定箱', value: '1' }
          ]
        },
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            align: 'center',
            customRender: function(t, r, index) {
              return (t.pageNo - 1) * t.pageSize + parseInt(index) + 1
            }
          }, {
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo',
            // fixed:'left',
            width: 80
          }, {
            title: '箱号',
            align: 'center',
            // dataIndex: 'boxNo',
            scopedSlots: { customRender: 'boxNOAutoHoldDesc' }
          }, {
            title: '箱主',
            align: 'center',
            dataIndex: 'boxOwner'
          }, {
            title: '箱型',
            align: 'center',
            dataIndex: 'boxType'
          },
          {
            title: '龙潭重箱进场时间',
            align: 'center',
            dataIndex: 'inYardTime'
          }, {
            title: '南京运抵发送时间',
            align: 'center',
            dataIndex: 'njArrivalSendTime'
          }, {
            title: '南京运抵回执时间',
            align: 'center',
            dataIndex: 'njArrivalReceiptTime'
          }, {
            title: '南京海关放行时间',
            align: 'center',
            dataIndex: 'njCustomReleaseTime'
          },
          {
            title: '查验时间',
            align: 'center',
            dataIndex: 'njCustomCheckTime'
          }, {
            title: '出口船名',
            align: 'center',
            dataIndex: 'englishVessel'
          }, {
            title: '出口航次',
            align: 'center',
            dataIndex: 'leaveVoyno'
          }, {
            title: '码头',
            align: 'center',
            dataIndex: 'dockCode'
          }, {
            title: '进场时间',
            align: 'center',
            dataIndex: 'arrivalPortTime'
          }, {
            title: '进场方式',
            align: 'center',
            dataIndex: 'arrivalSpaceType'
          }, {
            title: '出场方式',
            align: 'center',
            dataIndex: 'leaveSpaceType'
          }, {
            title: '运抵报文时间',
            align: 'center',
            dataIndex: 'arrivalMessageSendTime'
          }, {
            title: '上海海关放行时间',
            align: 'center',
            dataIndex: 'shCustomReleaseTime'
          }, {
            title: '码头放行',
            align: 'center',
            dataIndex: 'portReleaseType'
          }, {
            title: '码头配载',
            align: 'center',
            dataIndex: 'stowageType'
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            fixed: 'right',
            width: 80
          }],
        dataSource: [],
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['5', '10', '20'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        isorter: {
          column: 'createTime',
          order: 'desc'
        },
        selectedRowKeys: [],
        selectionRows: [],
        visible: false,
        toggleSearchStatus: false
      }
    },
    created() {
      this.resetScreenSize()
    },
    methods: {
      seeTip: function() {
        this.seen = true
        // alert('你好')
      },
      disSeeTip: function() {
        this.seen = false
      },
      getlock(data) {
        console.log(data)
        this.loadData(data)
      },
      show() {
        this.visible = true
      },
      searchQuery() {
        this.loadData(1)
      },
      searchReset() {
        this.queryParam.billNo = undefined
        this.queryParam.boxNo = undefined
        this.jcheckbox.values = undefined
        this.loadData(1)
      },
      loadData(arg) {
        if (arg === 1) {
          this.ipagination.current = 1
        }

        let params = this.getQueryParams()//查询条件arg
        console.log('查询参数=', params)
        getGroupDetails(params).then((res) => {
          // console.log('点击详情=', res)
          if (res.success) {
            this.dataSource = res.result.records
            this.dataSource.forEach(record => {
              record.pageNo = this.ipagination.current
              record.pageSize = this.ipagination.pageSize
            })
            this.ipagination.total = res.result.total
            this.visible = true
          }
        })
      },
      getQueryParams() {
        let param = Object.assign({}, this.queryParam, this.isorter, { autoHoldDesc: this.jcheckbox.values })
        param.pageNo = this.ipagination.current
        param.pageSize = this.ipagination.pageSize
        return filterObj(param)
      },
      getQueryField() {
        let str = 'id,'
        for (let a = 0; a < this.columns.length; a++) {
          str += ',' + this.columns[a].dataIndex
        }
        return str
      },
      onSelectChange(selectedRowKeys) {
        this.selectedRowKeys = selectedRowKeys
      },
      handleTableChange(pagination, filters, sorter) {
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field
          this.isorter.order = 'ascend' === sorter.order ? 'asc' : 'desc'
        }
        this.ipagination = pagination
        this.loadData()
      },
      onSelect(record, selected) {
        if (selected == true) {
          this.selectionRows.push(record)
        } else {
          this.selectionRows.forEach(function(item, index, arr) {
            if (item.id == record.id) {
              arr.splice(index, 1)
            }
          })
        }
      },
      handleCancel() {
        this.dataSource = []
        this.ipagination.total = 0
        this.selectionRows = []
        this.selectedRowKeys = []
        this.visible = false
      },
      handleOk() {
        this.$emit('choseUser', this.selectionRows)
        this.handleCancel()
      },
      searchByquery() {
        this.loadData(1)
      },
      handleToggleSearch() {
        this.toggleSearchStatus = !this.toggleSearchStatus
      },
      resetScreenSize() {
        let screenWidth = document.body.clientWidth
        if (screenWidth < 500) {
          this.scrollTrigger = { x: 800 }
        } else {
          this.scrollTrigger = {}
        }
      },
      viewTask(record) {
        this.$refs.taskInfoModal.loadTaskInfo(record.taskId)
        this.$refs.taskInfoModal.title = '任务详情'
        this.$refs.taskInfoModal.show()
      },
      taskInfoModalOk() {

      },
      viewGroupDetail(record, filterColumn) {
        this.$refs.groupDetailModal.resetScreenSize()
        this.$refs.groupDetailModal.queryParam = { groupId: record.id, filterColumn: filterColumn }
        this.$refs.groupDetailModal.ipagination.current = 1
        this.$refs.groupDetailModal.loadData()
        this.$refs.groupDetailModal.title = '提单追踪明细'
      }


    }
  }
</script>
<style scoped>
  .ant-card-body .table-operator {
    margin-bottom: 18px;
  }

  .ant-table-tbody .ant-table-row td {
    padding-top: 15px;
    padding-bottom: 15px;
  }

  .anty-row-operator button {
    margin: 0 5px
  }

  .ant-btn-danger {
    background-color: #ffffff
  }

  .ant-modal-cust-warp {
    height: 100%
  }

  .ant-modal-cust-warp .ant-modal-body {
    height: calc(100% - 110px) !important;
    overflow-y: auto
  }

  .ant-modal-cust-warp .ant-modal-content {
    height: 90% !important;
    overflow-y: hidden
  }

  .anty-img-wrap {
    height: 25px;
    position: relative;
  }

  .anty-img-wrap > img {
    max-height: 100%;
  }
</style>