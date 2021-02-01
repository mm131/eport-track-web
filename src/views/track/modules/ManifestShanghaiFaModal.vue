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
          <a-col :span="8">
            <a-form-item label="提单号">
              <a-input placeholder="请输入提单号" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item label="二程船名">
              <a-input placeholder="请输入二程船名" v-model="queryParam.lineName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <a-form-item label="二程船名">
              <a-input placeholder="请输入二程船名" v-model="queryParam.lineVoyage"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>

    </div>


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
          <a href="javascript:;" @click="viewManifest(record)">查看</a>
        </span>
      </a-table>

    </div>

  </a-modal>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'

  export default {
    name: 'ManifestShanghaiFaModal',
    mixins: [JeecgListMixin],
    components: {
      JInput
    },
    data() {
      return {
        visible: false,
        queryParam: {},
        shipNameVoyage:"",
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
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '舱单状态',
            align: 'center',
            dataIndex: 'billStatus'
          },
          {
            title: '二程船名',
            align: 'center',
            dataIndex: 'lineName'
          },
          {
            title: '二程航次',
            align: 'center',
            dataIndex: 'lineVoyage'
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            fixed: 'right',
            width: 180
          }
        ],
        url: {
          list: '/fa/getFaContentList'
        }
      }
    },

    methods: {
      loadFaContaInfo(entity) {
        this.queryParam.shipNameVoyage = entity.shipNameVoyage;
        this.shipNameVoyage=entity.shipNameVoyage;
        this.loadData();
        this.visible = true;
      },

      searchQuery(){
        this.loadData(1);
      },

      searchReset(){
        this.queryParam={};
        this.queryParam.shipNameVoyage=this.shipNameVoyage
        this.loadData(1);
      },

      handleOk() {
        this.visible = false
      },

      viewManifest(record){
        this.$router.push({ path:'/track/manifest'  })
      }
    }


  }
</script>

<style scoped>

</style>