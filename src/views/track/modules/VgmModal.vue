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
      </a-table>

    </div>

  </a-modal>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'

  export default {
    name: 'VgmModal',
    mixins: [JeecgListMixin],
    components: {
      JInput
    },
    data() {
      return {
        visible: false,
        queryParam: {},
        title() {
          return 'vgm数据'
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
            dataIndex: 'billNo'
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'containerNo'
          },
          {
            title: '箱型',
            align: 'center',
            dataIndex: 'containerTypeSize'
          },
          {
            title: '封号 ',
            align: 'center',
            dataIndex: 'sealNo'
          },
          {
            title: 'VGM(KG)',
            align: 'center',
            dataIndex: 'vgmData'
          },
          {
            title: '称重方式',
            align: 'center',
            dataIndex: 'weightMethod'
          },
          {
            title: '船名',
            align: 'center',
            dataIndex: 'shipName'
          },
          {
            title: '航次',
            align: 'center',
            dataIndex: 'voyage'
          },
          {
            title: '码头',
            align: 'center',
            width: 150,
            dataIndex: 'terminal'
          },
          {
            title: '签名',
            align: 'center',
            dataIndex: 'signature'
          },
          {
            title: '发货人电话',
            align: 'center',
            dataIndex: 'telephone'
          },
          {
            title: '发货人邮箱',
            align: 'center',
            dataIndex: 'email'
          },
          {
            title: '发货联系人',
            align: 'center',
            dataIndex: 'contactPerson'
          },
          {
            title: '授权人',
            align: 'center',
            dataIndex: 'authorizedPerson'
          },
          {
            title: '发货人',
            align: 'center',
            dataIndex: 'shipperName'
          }
        ],
        url: {
          list: '/vgm/getVgmContent'
        }
      }
    },

    methods: {
      loadVgmInfo(entity){
        this.queryParam.id=entity.id;
        this.loadData();
        this.visible = true;
      },
      handleOk(){
        this.visible=false;
      }
    }


  }
</script>

<style scoped>

</style>