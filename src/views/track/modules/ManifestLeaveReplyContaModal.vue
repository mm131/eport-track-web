<template>

  <a-modal
    :title="title"
    :width="1000"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭">
    <div>
      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :loading="loading"
        @change="handleTableChange">

        <span slot="chkResult" slot-scope="text, record">
          <span v-if="record.chkFlag==='01'">审核通过</span>
          <span v-else-if="record.chkFlag==='03'">退单</span>
          <span v-else></span>
        </span>
      </a-table>

    </div>

  </a-modal>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'

  export default {
    name: 'ManifestLeaveReplyContaModal',
    mixins: [JeecgListMixin],
    components: {
      JInput
    },
    data() {
      return {
        visible: false,
        queryParam: {},
        title() {
          return '集装箱列表'
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
            title: '运输工具名称',
            align: 'center',
            dataIndex: 'shipName'
          },
          {
            title: '航次',
            align: 'center',
            dataIndex: 'voyageNo'
          },
          {
            title: '集装箱号',
            align: 'center',
            dataIndex: 'contaNo'
          },
          {
            title: '放行状态',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'chkResult' }
          },
          {
            title: '备注',
            align: 'center',
            dataIndex: 'notes'
          }
        ],
        url: {
          list: '/manifestNanjing/getLeaveReplyContaList'
        }
      }
    },

    methods: {
      loadManifestLeaveReplyConta(entity) {
        this.queryParam.id = entity.id
        this.loadData()
        this.visible = true
      },
      handleOk() {
        this.visible = false
      }
    }


  }
</script>

<style scoped>

</style>