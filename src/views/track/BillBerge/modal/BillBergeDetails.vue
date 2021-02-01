<template>
  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭">

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
  import { getBargeDetails } from '@/api/api'
  import { filterObj } from '@/utils/util'

  export default {
    name: 'BillBergeDetails',
    mixins: [JeecgListMixin],
    data() {
      return {
        title() {
          return ''
        },
        visible: false,
        dataSource: [],
        loading: false,
        result: {},
        isorter: {
          column: '',
          order: ''
        },
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
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
            title: '船名',
            align: 'center',
            dataIndex: 'outShipName'
          }, {
            title: '航次',
            align: 'center',
            dataIndex: 'exVoyage'
          }, {
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo'
          }, {
            title: '箱型',
            align: 'center',
            dataIndex: 'boxType'
          }, {
            title: '进场时间',
            align: 'center',
            dataIndex: 'inYardTime'
          },
          {
            title: '南京运抵发送时间',
            align: 'center',
            dataIndex: 'njArrivalSendTime'
          }, {
            title: '南京运抵回执时间',
            align: 'center',
            dataIndex: 'njArrivalReceiptTime'
          }
        ],
        url: {
          list: '/BargeManage/task/details'
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
      load(record) {
        console.log(record.outShipName, record.exVoyage)
        this.result = record
        let params = this.getQueryParams(record.outShipName, record.exVoyage)//查询条件
        console.log('参数为：', params)
        getBargeDetails(params).then((res) => {
          console.log('驳船管理详情接口结果为：', res)
          if (res.success) {
            this.dataSource = res.result.records
            this.ipagination.pageNo = res.result.pages
            this.ipagination.pageSize = res.result.size
            this.ipagination.total = res.result.total
          }
        })
      },
      getQueryParams(outShipName, exVoyage) {
        let param = Object.assign({}, this.queryParam, this.isorter, { outShipName: outShipName, exVoyage: exVoyage })
        param.pageNo = this.ipagination.current
        param.pageSize = this.ipagination.pageSize
        return filterObj(param)
      },
      handleTableChange(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field
          this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
        }
        this.ipagination = pagination
        this.load(this.result)
      }
    }
  }
</script>

<style scoped>

</style>