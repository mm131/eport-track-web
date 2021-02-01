<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="船名">
              <a-input placeholder="输入船名" v-model="queryParam.shipName"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searcShip" icon="search">查询</a-button>
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
        rowKey="shipNO"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        :scroll="{ x: 'max-content' }"
        @change="handleTableChange">

        <span slot="lengthRender" slot-scope="text, record">
          {{record.length / 10}}米
        </span>
        <span slot="widthRender" slot-scope="text, record">
          {{record.width / 10}}米
        </span>
        <span slot="draughtRender" slot-scope="text, record">
          {{record.draught / 1000}}米
        </span>


<!--        <span slot="action" slot-scope="text, record">-->
<!--          <a href="javascript:;"  @click="viewInfo(record)">详情</a>-->
<!--        </span>-->
      </a-table>
    </div>

<!--    <shipment-info-modal ref="ShipInfoModal"></shipment-info-modal>-->
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  // import ShipInfoModal from './modules/ShipInfoModal'

  export default {
    name: 'ShipQueryList',
    mixins: [JeecgListMixin],
    components: {
      JInput,
      // ShipInfoModal
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        queryParam: {},
        columns: [
          {
            title: '船名',
            align: 'center',
            dataIndex: 'name'
          },
          {
            title: 'MMSI',
            align: 'center',
            dataIndex: 'mmsi'
          },
          {
            title: 'IMO号',
            align: 'center',
            dataIndex: 'imo'
          },
          {
            title: '呼号 ',
            align: 'center',
            dataIndex: 'callsign'
          },
          {
            title: '船长',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'lengthRender' }
          },
          {
            title: '船宽',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'widthRender' }
          },
          {
            title: '吃水',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'draughtRender' }
          },
          {
            title: '目的地',
            align: 'center',
            dataIndex: 'dest'
          }
        ],
        ipagination:{
          current: 1,
          pageSize: 10000,
          pageSizeOptions: ['1000'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: '/ship/shipInfo'
        }
      }
    },
    mounted() {
    },

    methods: {
      searcShip() {
        this.searchQuery();
      },

      // viewInfo(record){
      //   this.$refs.ShipInfoModal.loadShipInfo(record)
      //   this.$refs.ShipInfoModal.title = '船舶详情'
      //   this.$refs.ShipInfoModal.show()
      // },
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>