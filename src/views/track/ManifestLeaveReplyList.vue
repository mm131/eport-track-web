<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" >
        <a-row :gutter="24">

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
            <a-form-item label="箱号">
              <a-input placeholder="" v-model="queryParam.contaNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="审核状态">
              <a-select v-model="queryParam.chkFlag" placeholder="请选择">
                <a-select-option value='01'>审核通过</a-select-option>
                <a-select-option value='03'>退单</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>


          <a-col :md="6" :sm="12">
            <a-form-item label="离场时间">
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
              <a-button type="primary"  @click="batchDownload" icon="export" style="margin-left: 8px">下载</a-button>
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
          <a href="javascript:;" @click="getManifestLeaveReplyConta(record)">集装箱信息</a>
        </span>

        <span slot="chkResult" slot-scope="text, record">
          <span v-if="record.chkFlag==='01'">审核通过</span>
          <span v-else-if="record.chkFlag==='03'">退单</span>
          <span v-else></span>
        </span>

      </a-table>
    </div>

    <manifest-leave-reply-conta-modal ref="ManifestLeaveReplyContaModal"></manifest-leave-reply-conta-modal>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import moment from 'moment'
  import ManifestLeaveReplyContaModal from './modules/ManifestLeaveReplyContaModal'
  Vue.prototype.$moment = moment

  export default {
    name: 'ManifestLeaveReplyList',
    mixins: [JeecgListMixin],
    components: {
      ManifestLeaveReplyContaModal,
      JInput,
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        startValue: null,
        endValue: null,
        endOpen: false,
        selectedRowKeys: [],
        hasSelected: false,
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
            title: '航次/班 ',
            align: 'center',
            dataIndex: 'voyageNo'
          },
          {
            title: '放行状态',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'chkResult' }
          },
          {
            title: '离场时间',
            align: 'center',
            dataIndex: 'messageCreateTime'
          },
          {
            title: '备注',
            align: 'center',
            dataIndex: 'notes'
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
          list: '/manifestNanjing/getLeaveReplyList',
          downloadUrl: 'manifestNanjing/download'
        }
      }
    },

    mounted() {
      var date1=new Date().toLocaleDateString()+' 00:00:00';
      var date2=new Date().toLocaleDateString()+' 23:59:59';
      this.startValue=moment(date1,'YYYY-MM-DD HH:mm:ss');
      this.endValue=moment(date2,'YYYY-MM-DD HH:mm:ss');
      this.queryParam.startDate=this.$moment(this.startValue).format('YYYY-MM-DD HH:mm:ss');
      this.queryParam.endDate=this.$moment(this.endValue).format('YYYY-MM-DD HH:mm:ss');

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
        if(this.startValue==null || this.endValue==null){
          this.$message.error('请选离场开始、结束时间')
          return;
        }
        this.loadData(this.queryParam);
      },

      searchReset1() {
        var date1=new Date().toLocaleDateString()+' 00:00:00';
        var date2=new Date().toLocaleDateString()+' 23:59:59';
        this.startValue=moment(date1,'YYYY-MM-DD HH:mm:ss');
        this.endValue=moment(date2,'YYYY-MM-DD HH:mm:ss');
        this.queryParam = {};
        this.loadData(1)
      },

      getManifestLeaveReplyConta(record) {
        this.$refs.ManifestLeaveReplyContaModal.loadManifestLeaveReplyConta(record)
      },

      onSelectChange(selectedRowKeys) {
        this.selectedRowKeys = selectedRowKeys;
        if(this.selectedRowKeys.length>0){
          this.hasSelected=true;
        }else{
          this.hasSelected=false;
        }
      },


      batchDownload(){
        var start = new Date(this.startValue);
        var end = new Date(this.endValue);
        var days = end.getTime()-start.getTime();
        var day = parseInt(days / (1000 * 60 * 60 * 24));
        console.log(day);
        if(day > 30){
          this.$message.error('只能下载开始、结束时间间隔小于30天的离场记录')
          return;
        }

        let url=`${window._CONFIG['domianURL']}/${this.url.downloadUrl}`;
        window.location.href=url;
        this.selectedRowKeys=[];
        this.hasSelected=false;
      }


    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>