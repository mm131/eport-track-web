<template>

  <a-modal
    :title="title"
    :width="1200"
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
        :scroll="{ x: 'max-content' }"
        @change="handleTableChange">

        <span slot="modifyRecord" slot-scope="text, record">
          <span v-if="record.modifyNo===0">原始数据</span>
          <span v-else >
            第{{record.modifyNo}}次修改
          </span>
        </span>

      </a-table>
    </div>
  </a-modal>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { getAction } from '@/api/manage'

  export default {
    name: 'ShipmentFormRecordModal',
    mixins: [JeecgListMixin],
    components: {
      JInput
    },
    data() {
      return {
        visible: false,
        queryParam: {},
        dataSource:[],
        title() {
          return '修改记录'
        },

        columns: [
          {
            title: '#',
            align: 'center',
            dataIndex: 'rowIndex',
            customRender: function (text, r, index) {
              return (text !== '合计') ? (parseInt(index) + 1) : text
            }
          },
          {
            title: '修改记录',
            align: 'center',
            width: 120,
            dataIndex: '',
            scopedSlots: { customRender: 'modifyRecord' }
          },
          {
            title: '修改人',
            align: 'center',
            dataIndex: 'updateBy'
          },
          {
            title: '修改时间',
            align: 'center',
            dataIndex: 'updateTime'
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
            title: '封签',
            align: 'center',
            dataIndex: 'seal'
          },
          {
            title: '尺寸 ',
            align: 'center',
            dataIndex: 'boxSize'
          },
          {
            title: 'FEL',
            align: 'center',
            dataIndex: 'fel'
          },
          {
            title: '英文货名',
            align: 'center',
            dataIndex: 'goodsNameEn'
          },
          {
            title: '中文货名',
            align: 'center',
            dataIndex: 'goodsNameCh'
          },
          {
            title: '唛头',
            align: 'center',
            dataIndex: 'label'
          },
          {
            title: '包装',
            align: 'center',
            dataIndex: 'packaging'
          },
          {
            title: '重量',
            align: 'center',
            dataIndex: 'weight'
          },
          {
            title: '件数',
            align: 'center',
            dataIndex: 'number'
          },
          {
            title: '体积',
            align: 'center',
            dataIndex: 'volume'
          },
          {
            title: '箱主',
            align: 'center',
            dataIndex: 'boxOwner'
          },
          {
            title: '干线船名',
            align: 'center',
            dataIndex: 'lineName'
          },
          {
            title: '干线航次',
            align: 'center',
            dataIndex: 'lineVoyage'
          },
          {
            title: '干线地点',
            align: 'center',
            dataIndex: 'lineLocation'
          },
          {
            title: '干线船船期',
            align: 'center',
            dataIndex: 'lineDate'
          },
          {
            title: '卸货港',
            align: 'center',
            dataIndex: 'unloadPort'
          },
          {
            title: '目的港',
            align: 'center',
            dataIndex: 'destinationPort'
          },
          {
            title: '中转港代码',
            align: 'center',
            dataIndex: 'transitPortCode'
          },
          {
            title: 'VGM',
            align: 'center',
            dataIndex: 'vgm'
          },
          {
            title: 'VGM称重方式',
            align: 'center',
            dataIndex: 'vgmWeightMethod'
          }
        ],
      }
    },

    methods: {
      modifyRecord(record){
        getAction(`/shipmentForm/getShipmentFormContentRecord`,{id:record.id}).then(res=>{
          console.log(res);
          if(res.success){
            this.dataSource = res.result;
          }
        })
        this.visible = true;
      },
      handleOk(){
        this.visible=false;
      },
    }
  }
</script>

<style scoped>

</style>