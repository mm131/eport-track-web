<template>
  <a-card :bordered="false">
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

          <span slot="action" slot-scope="text, record">
            <a href="javascript:;" @click="send(record)">补发</a>
          </span>
        </a-table>
      </div>
    </a-modal>

    <manifest-nanjing-conta-code-modal ref="manifestNanjingContaCodeModal"/>

  </a-card>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { postAction } from '@/api/manage'
  import manifestNanjingContaCodeModal from './ManifestNanjingContaCodeModal'


  export default {
    name: 'ManifestNanjingContaModal',
    mixins: [JeecgListMixin],
    components: {
      JInput,
      manifestNanjingContaCodeModal
    },
    data() {
      return {
        visible: false,
        queryParam: {},
        title() {
          return '箱号列表'
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
            title: '运输工具编号',
            align: 'center',
            dataIndex: 'shipId'
          },
          {
            title: '运输工具名称',
            align: 'center',
            dataIndex: 'shipName'
          },
          {
            title: '航次/班 ',
            align: 'center',
            dataIndex: 'voyageNo'
          },
          {
            title: '提运单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'contaId'
          },
          {
            title: '运抵报告状态',
            align: 'center',
            dataIndex: 'njArrivalReceipt'
          },
          {
            title: '运抵补发',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            width: 100
          }
        ],
        url: {
          list: '/manifestNanjing/getManifestConta'
        }
      }
    },

    methods: {
      send(record) {
        this.$refs.manifestNanjingContaCodeModal.show(record)
        this.$refs.manifestNanjingContaCodeModal.getCaptchaImg()
        this.$refs.manifestNanjingContaCodeModal.cleanCode()


      },
      loadManifestContaInfo(entity) {
        this.queryParam.id = entity.id
        this.loadData()
        this.visible = true
      }
      ,
      handleOk() {
        this.visible = false
      }
    }


  }
</script>

<style scoped>

</style>