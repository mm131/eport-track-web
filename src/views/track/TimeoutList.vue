<template>
  <a-card :bordered="false">
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="12">
            <a-form-item label="箱号">
              <a-input placeholder="输入箱号查询" v-model="queryParam.boxNo"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="12">
            <a-form-item label="船名">
              <a-input placeholder="输入船名查询" v-model="queryParam.njShipName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="12">
            <a-form-item label="航次">
              <a-input placeholder="输入航次查询" v-model="queryParam.njVoyageNo"></a-input>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row :gutter="24">
          <a-col :md="6" :sm="12">
            <a-form-item label="超时类型">
              <a-select placeholder="请选择" default-value="DEPARTURE" v-model="queryParam.timeoutType" @change="handleTypeChange">
                <a-select-option value="DEPARTURE">用箱超时</a-select-option>
                <a-select-option value="IN_YARD">堆存超时</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset2" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <a-table
      ref="table"
      size="middle"
      bordered
      rowKey="id"
      :columns="columns"
      :dataSource="dataSource"
      :pagination="ipagination"
      :loading="loading"
      :scroll="{ x: 'max-content' }"
      @change="handleTableChange">

        <span slot="entryTitle" slot-scope="text, record">
          {{ getEntryTitle(record) }}
        </span>

        <span slot="daysExpireAction" slot-scope="text, record">
          <div>
            <span v-bind:style="{ color: 'red' }">{{ record.expireDays - 7}}</span><span>&nbsp;天</span>
          </div>
        </span>
    </a-table>
  </a-card>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { getAction, postAction } from '@/api/manage'

  export default {
    name: 'TimeoutList',
    components: {},
    mixins: [JeecgListMixin],
    data() {
      return {
        description: '超时查询',
        queryParam: {
          billNo: '',
          boxNo: '',
          timeoutType: 'DEPARTURE',
          njShipName: '',
          njVoyageNo: ''
        },
        isorter:
          {
            column: '',
            order: ''
          },
        loading: true,
        selectedRowKeys: [],
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
            title: '箱号',
            align: 'center',
            dataIndex: 'boxNo'
          },
          {
            title: '箱型',
            align: 'center',
            dataIndex: '',
            customRender: (record) => record.boxSize + record.boxType
          },
          {
            title: '出门时间',
            align: 'center',
            dataIndex: 'njDepartureTime'
          },
          {
            // title: '123',
            align: 'center',
            dataIndex: 'njEntryTime',
            scopedSlots: { title: 'entryTitle'}
          },
          {
            title: '装船时间',
            align: 'center',
            dataIndex: 'njLoadTime'
          },
          {
            title: '船名',
            align: 'center',
            dataIndex: 'njShipName'
          },
          {
            title: '航次',
            align: 'center',
            dataIndex: 'njVoyageNo'
          },
          {
            title: '超期天数',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'daysExpireAction' }
          }
        ],
        url: {
          list: '/timeout/list'
        },
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '50', '100'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        }
      }
    },
    methods: {
      searchReset2: function() {
        this.queryParam = {
          billNo: '',
          boxNo: '',
          timeoutType: 'DEPARTURE',
          njShipName: '',
          njVoyageNo: ''
        }
        this.loadData()
      },
      getEntryTitle: function() {
        return this.queryParam.timeoutType === 'DEPARTURE' ? '进门时间' : '进场时间';
      },
      handleTypeChange: function(value) {
        this.loadData()
      }
    }
  }
</script>

<style scoped>

</style>