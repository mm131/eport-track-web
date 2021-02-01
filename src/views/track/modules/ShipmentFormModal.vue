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
        <template
          v-for="col in ['billNo', 'boxNo', 'seal', 'boxSize', 'fel', 'goodsNameEn', 'goodsNameCh', 'label', 'packaging',
          'weight', 'number', 'volume', 'boxOwner', 'lineName', 'lineVoyage', 'lineLocation', 'lineDate', 'unloadPort',
          'destinationPort', 'transitPortCode', 'vgm', 'vgmWeightMethod']"
          :slot="col"
          slot-scope="text, record, index"
        >
          <div :key="col">
            <a-input
              v-if="record.editable"
              style="margin: -5px 0"
              :value="text"
              @change="e => handleChange(e.target.value, record.id, col)"
            />
            <template v-else>
              {{ text }}
            </template>
          </div>
        </template>
        <template slot="action" slot-scope="text, record, index">
          <div>
            <span v-if="record.editable">
              <a @click="() => save(record.id)">保存</a>
              <a-divider type="vertical" />
              <a-popconfirm title="确定取消吗?" @confirm="() => cancel()">
                <a>取消</a>
              </a-popconfirm>
            </span>
            <span v-else>
              <a :disabled="editingKey !== ''" @click="() => edit(record.id)">修改</a>
              <span v-if="record.modifyNo > 0">
                <a-divider type="vertical" />
                <a href="javascript:;"  @click="modifyRecord(record)">修改记录</a>
              </span>
            </span>
          </div>
        </template>
      </a-table>
    </div>
    <shipment-form-record-modal ref="ShipmentFormRecordModal" ></shipment-form-record-modal>

  </a-modal>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { postAction } from '@/api/manage'
  import ShipmentFormRecordModal from './ShipmentFormRecordModal'

  export default {
    name: 'ShipmentFormModal',
    mixins: [JeecgListMixin],
    components: {
      ShipmentFormRecordModal,
      JInput
    },
    data() {
      return {
        visible: false,
        queryParam: {},
        editingKey: '',
        title() {
          return '出运表数据'
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
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo',
            scopedSlots: { customRender: 'billNo' }
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'boxNo',
            scopedSlots: { customRender: 'boxNo' }
          },
          {
            title: '封签',
            align: 'center',
            dataIndex: 'seal',
            scopedSlots: { customRender: 'seal' }
          },
          {
            title: '尺寸 ',
            align: 'center',
            dataIndex: 'boxSize',
            scopedSlots: { customRender: 'boxSize' }
          },
          {
            title: 'FEL',
            align: 'center',
            dataIndex: 'fel',
            scopedSlots: { customRender: 'fel' }
          },
          {
            title: '英文货名',
            align: 'center',
            dataIndex: 'goodsNameEn',
            scopedSlots: { customRender: 'goodsNameEn' }
          },
          {
            title: '中文货名',
            align: 'center',
            dataIndex: 'goodsNameCh',
            scopedSlots: { customRender: 'goodsNameCh' }
          },
          {
            title: '唛头',
            align: 'center',
            dataIndex: 'label',
            scopedSlots: { customRender: 'label' }
          },
          {
            title: '包装',
            align: 'center',
            dataIndex: 'packaging',
            scopedSlots: { customRender: 'packaging' }
          },
          {
            title: '重量',
            align: 'center',
            dataIndex: 'weight',
            scopedSlots: { customRender: 'weight' }
          },
          {
            title: '件数',
            align: 'center',
            dataIndex: 'number',
            scopedSlots: { customRender: 'number' }
          },
          {
            title: '体积',
            align: 'center',
            dataIndex: 'volume',
            scopedSlots: { customRender: 'volume' }
          },
          {
            title: '箱主',
            align: 'center',
            dataIndex: 'boxOwner',
            scopedSlots: { customRender: 'boxOwner' }
          },
          {
            title: '干线船名',
            align: 'center',
            dataIndex: 'lineName',
            scopedSlots: { customRender: 'lineName' }
          },
          {
            title: '干线航次',
            align: 'center',
            dataIndex: 'lineVoyage',
            scopedSlots: { customRender: 'lineVoyage' }
          },
          {
            title: '干线地点',
            align: 'center',
            dataIndex: 'lineLocation',
            scopedSlots: { customRender: 'lineLocation' }
          },
          {
            title: '干线船船期',
            align: 'center',
            dataIndex: 'lineDate',
            scopedSlots: { customRender: 'lineDate' }
          },
          {
            title: '卸货港',
            align: 'center',
            dataIndex: 'unloadPort',
            scopedSlots: { customRender: 'unloadPort' }
          },
          {
            title: '目的港',
            align: 'center',
            dataIndex: 'destinationPort',
            scopedSlots: { customRender: 'destinationPort' }
          },
          {
            title: '中转港代码',
            align: 'center',
            dataIndex: 'transitPortCode',
            scopedSlots: { customRender: 'transitPortCode' }
          },
          {
            title: 'VGM',
            align: 'center',
            dataIndex: 'vgm',
            scopedSlots: { customRender: 'vgm' }
          },
          {
            title: 'VGM称重方式',
            align: 'center',
            dataIndex: 'vgmWeightMethod',
            scopedSlots: { customRender: 'vgmWeightMethod' }
          },
          {
            title: '操作',
            align: 'center',
            dataIndex: 'action',
            fixed:'right',
            width: 150,
            scopedSlots: { customRender: 'action' }
          },
        ],
        url: {
          list: '/shipmentForm/getShipmentFormContent'
        }
      }
    },

    methods: {
      loadShipmentFormInfo(entity){
        this.queryParam.id=entity.id;
        this.loadData();
        this.editingKey='';
        this.visible = true;
      },
      handleOk(){
        this.visible=false;
      },

      handleChange(value, id, column) {
        const newData = [...this.dataSource];
        const target = newData.filter(item => id === item.id)[0];
        if (target) {
          target[column] = value;
          this.dataSource = newData;
        }
      },
      edit(id) {
        const newData = [...this.dataSource];
        const target = newData.filter(item => id === item.id)[0];
        this.editingKey = id;
        if (target) {
          target.editable = true;
          this.dataSource = newData;
        }
      },

      save(id) {
        const newData = [...this.dataSource];
        const target = newData.filter(item => id === item.id)[0];
        if (target) {
          postAction(`/shipmentForm/updateShipmentFormContent`,target).then(res=>{
            console.log(res);
            if(res.success){
              delete target.editable;
              this.dataSource = newData;
              this.loadData();
              this.$message.success("修改成功");
            }else{
              this.loadData();
              this.$message.error("修改失败");
            }
            this.editingKey = '';
          })
        }

      },

      cancel() {
        this.editingKey = '';
        this.loadData();
      },

      modifyRecord(record){
        this.$refs.ShipmentFormRecordModal.modifyRecord(record)
      }


    }
  }
</script>

<style scoped>

</style>