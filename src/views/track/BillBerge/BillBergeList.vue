<template>
  <a-card :bordered="false">
    <a-table
      ref="table"
      size="middle"
      bordered
      rowKey="id"
      :columns="columns"
      :dataSource="dataSource"
      :pagination="ipagination"
      :loading="loading"
      @change="handleTableChange">
        <span slot="action" slot-scope="text, record">
            <a href="javascript:;" @click="searchDetail(record)">详情</a>
        </span>
    </a-table>
    <berge-detail-modal ref="bergeDetailModal"/>
  </a-card>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import bergeDetailModal from './modal/BillBergeDetails'

  export default {
    name: 'BillBergeList',
    mixins: [JeecgListMixin],
    components: {
      bergeDetailModal
    },
    data() {
      return {
        description: '装船超时管理页面',
        loading: false,
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
            title: '船名',
            align: 'center',
            dataIndex: 'outShipName'
          },
          {
            title: '航次',
            align: 'center',
            width: 100,
            dataIndex: 'exVoyage'
          },
          {
            title: '提单数',
            align: 'center',
            dataIndex: 'billNoCount'
          },
          {
            title: '箱数',
            align: 'center',
            dataIndex: 'boxNoCount'
          },
          {
            title: '未放行数',
            align: 'center',
            dataIndex: 'njCustomReleaseTimeNullCount'
          },
          {
            title: '未装船数',
            align: 'center',
            dataIndex: 'njLoadTimeNullCount'
          },
          {
            title: '存放超期箱数',
            align: 'center',
            dataIndex: 'outTimeFlagIsOneCount',
            scopedSlots: { customRender: 'out' }
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
          list: '/BargeManage/task/list'
        }
      }
    },
    methods: {
      searchDetail(record) {
        this.$refs.bergeDetailModal.title = '详情'
        this.$refs.bergeDetailModal.show()
        this.$refs.bergeDetailModal.load(record)
      }

    }
  }
</script>

<style scoped>

</style>