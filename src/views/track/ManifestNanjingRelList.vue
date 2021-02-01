<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" >
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="进出口">
              <a-select v-model="queryParam.ieFlag" placeholder="全部">
                <a-select-option value="I">进口</a-select-option>
                <a-select-option value="E">出口</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="运输工具名称">
              <a-input placeholder="" v-model="queryParam.shipName"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="航次">
              <a-input placeholder="" v-model="queryParam.voyageNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="提运单号">
              <a-input placeholder="" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="入库时间">
              <a-date-picker
                v-model="startValue"
                :disabled-date="disabledStartDate"
                show-time
                format="YYYY-MM-DD HH:mm:ss"
                placeholder="开始时间"
                @openChange="handleStartOpenChange"
              />
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="">
              <a-date-picker
                v-model="endValue"
                :disabled-date="disabledEndDate"
                show-time
                format="YYYY-MM-DD HH:mm:ss"
                placeholder="结束时间"
                :open="endOpen"
                @openChange="handleEndOpenChange"
              />
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset1" icon="reload" style="margin-left: 8px">重置</a-button>
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
        :pagination="ipagination"
        :loading="loading"
        :scroll="{ x: 'max-content' }"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a href="javascript:;" @click="getManifestRelConta(record)">集装箱信息</a>
        </span>

        <span slot="ieFlagType" slot-scope="text, record">
          <span v-if="record.ieFlag==='I'">进口(I)</span>
          <span v-else-if="record.ieFlag==='E'">出口(E)</span>
          <span v-else></span>
        </span>

      </a-table>
    </div>

    <manifest-nanjing-rel-conta-modal ref="ManifestNanjingRelContaModal"></manifest-nanjing-rel-conta-modal>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import moment from 'moment'
  import ManifestNanjingRelContaModal from './modules/ManifestNanjingRelContaModal'
  Vue.prototype.$moment = moment

  export default {
    name: 'ManifestNanjingRelList',
    mixins: [JeecgListMixin],
    components: {
      ManifestNanjingRelContaModal,
      JInput,
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        startValue: null,
        endValue: null,
        endOpen: false,
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
            title: '进/出口',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'ieFlagType' }
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
            title: '放行状态',
            align: 'center',
            dataIndex: 'releaseShow'
          },
          {
            title: '放行时间',
            align: 'center',
            dataIndex: 'messageCreateTime'
          },
          {
            title: '操作',
            align: 'center',
            dataIndex: 'action',
            fixed: 'right',
            width: 200,
            scopedSlots: { customRender: 'action' }
          }
        ],
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['20'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: '/manifestNanjing/getNanjingRelList',
        }
      }
    },

    mounted() {
      var date1=new Date().toLocaleDateString()+' 00:00:00';
      var date2=new Date().toLocaleDateString()+' 23:59:59';
      this.startValue=moment(date1,'YYYY-MM-DD HH:mm:ss');
      this.endValue=moment(date2,'YYYY-MM-DD HH:mm:ss');
    },

    methods: {

      moment,

      disabledStartDate(startValue) {
        const endValue = this.endValue;
        if (!startValue || !endValue) {
          return false;
        }
        return startValue.valueOf() > endValue.valueOf();
      },
      disabledEndDate(endValue) {
        const startValue = this.startValue;
        if (!endValue || !startValue) {
          return false;
        }
        return startValue.valueOf() >= endValue.valueOf();
      },

      handleStartOpenChange(open) {
        if (!open) {
          this.endOpen = true;
        }
        this.queryParam.startDate=this.$moment(this.startValue).format('YYYY-MM-DD HH:mm:ss');
      },
      handleEndOpenChange(open) {
        this.endOpen = open;
        this.queryParam.endDate=this.$moment(this.endValue).format('YYYY-MM-DD HH:mm:ss');
      },

      searchQuery(){
        if(this.startValue!=null && this.endValue==null){
          // this.$message.error('请选择结束时间')
          // return;
          this.queryParam.endDate=null;
        }
        if(this.startValue==null && this.endValue!=null){
          this.$message.error('请选择开始时间')
          return;
        }
        if(this.startValue==null && this.endValue==null){
          this.queryParam.startDate=null;
          this.queryParam.endDate=null;
        }
        this.loadData(this.queryParam);
      },

      searchReset1() {
        var date1=new Date().toLocaleDateString()+' 00:00:00';
        var date2=new Date().toLocaleDateString()+' 23:59:59';
        this.startValue=moment(date1,'YYYY-MM-DD HH:mm:ss');
        this.endValue=moment(date2,'YYYY-MM-DD HH:mm:ss');
        this.queryParam = {}
        this.loadData(1)
      },

      getManifestRelConta(record) {
        this.$refs.ManifestNanjingRelContaModal.loadManifestRelContaInfo(record)
      },


    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>