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
        rowKey="boxNo"
        :columns="columns"
        :dataSource="dataSource"
        :loading="loading"
        @change="handleTableChange">

        <span slot="arrivalType" slot-scope="text, record">
          <span v-if="record.arrivalMessageSendTime!=null">Y</span>
          <span v-else>N</span>
        </span>

        <span slot="customType" slot-scope="text, record">
          <span v-if="record.customReleaseTime!=null">Y</span>
          <span v-else>N</span>
        </span>

      </a-table>

    </div>

  </a-modal>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'

  export default {
    name: 'ManifestContaModal',
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
            title: '箱号',
            align: 'center',
            dataIndex: 'boxNo'
          },
          {
            title: '箱型',
            align: 'center',
            dataIndex: 'boxSize'
          },
          {
            title: '持箱人',
            align: 'center',
            dataIndex: 'boxOperator'
          },
          {
            title: '箱状态',
            align: 'center',
            dataIndex: 'boxStatus'
          },
          {
            title: '进场方式',
            align: 'center',
            dataIndex: 'arrivalSpaceType'
          },
          {
            title: '运抵',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'arrivalType' }
          },
          {
            title: '海放',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'customType' }
          },
          {
            title: '码放',
            align: 'center',
            dataIndex: 'portReleaseType'
          }
        ],
        url: {
          list: '/manifest/getManifestConta'
        }
      }
    },

    methods: {
      loadContaInfo(entity) {
        this.queryParam.billNo = entity.billNo
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