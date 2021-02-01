<template>

  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭">

    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <a-form-item label="箱号">
              <a-input placeholder="输入箱数查询" v-model="queryParam.boxNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery()" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
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
      :pagination="ipagination"
      :dataSource="dataSource"
      :loading="loading"
      @change="handleTableChange">

    </a-table>

  </a-modal>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { getBillDetails } from '@/api/api'
  import { filterObj } from '@/utils/util'

  export default {
    name: 'GroupDetailModal',
    mixins: [JeecgListMixin],
    data() {
      return {
        queryParam: {},
        isorter: {
          column: '',
          order: ''
        },
        title: '',
        billNum: null,
        visible: false,
        dataSource: [],
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0,
          ccc: ''
        },
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
            title: '箱号',
            align: 'center',
            dataIndex: 'boxNo'
          },
          {
            title: '出门时间',
            align: 'center',
            dataIndex: 'njDepartureTime'
          },
          {
            title: '进门时间',
            align: 'center',
            dataIndex: 'njEntryTime'
          },
          {
            title: '装船时间',
            align: 'center',
            dataIndex: 'njLoadTime'
          },
          {
            title: '进场时间',
            align: 'center',
            dataIndex: 'inYardTime',
            customRender: function(t, r, index) {
              if (t === null || t === '' || t === undefined) {
                return r.njEntryTime
              } else {
                return t
              }
            }
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
          }
        ],
        url: {
          list: '/bill/task/details'
        }
      }
    },
    methods: {
      handleOk() {
        this.visible = false
      },
      show() {
        this.visible = true
      },
      searchQuery() {
        let params = this.queryParam
        getBillDetails(params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records
            this.ipagination.pageNo = res.result.pages
            this.ipagination.pageSize = res.result.size
            this.ipagination.total = res.result.total
          }
        })
      },
      loadData(arg, count, index) {
        console.log('点击箱数查询参数', arg, count, index)
        let params = this.getQueryParams(arg, count, index)//查询条件
        console.log('参数为：', params)
        getBillDetails(params).then((res) => {
          console.log('结果为：', res)
          if (res.success) {
            this.dataSource = res.result.records
            this.ipagination.pageNo = res.result.pages
            this.ipagination.pageSize = res.result.size
            this.ipagination.total = res.result.total
            //将提单号保存给handleTableChange事件使用
            this.ipagination.ccc = res.result.records[0].billNo
            //回写搜索框
            this.queryParam.billNo = res.result.records[0].billNo
          }
        })
      },
      getQueryParams(aaa, count, index) {
        let param = Object.assign({}, this.queryParam, this.isorter, { billNo: aaa, count: count, flag: index })
        param.pageNo = this.ipagination.current
        param.pageSize = this.ipagination.pageSize
        return filterObj(param)
      },
      searchReset() {
      },
      handleTableChange(pagination) {
        this.ipagination = pagination
        this.loadData(pagination.ccc)
      }
    }
  }
</script>

<style scoped>

</style>