<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="提运单号">
              <a-input placeholder="" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

<!--          <a-col :md="6" :sm="12">-->
<!--            <a-form-item label="船名">-->
<!--              <a-input placeholder="" v-model="queryParam.shipName"></a-input>-->
<!--            </a-form-item>-->
<!--          </a-col>-->

<!--          <a-col :md="6" :sm="12">-->
<!--            <a-form-item label="航次">-->
<!--              <a-input placeholder="" v-model="queryParam.voyageNo"></a-input>-->
<!--            </a-form-item>-->
<!--          </a-col>-->

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
      </a-table>
    </div>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import moment from 'moment'
  Vue.prototype.$moment = moment

  export default {
    name: 'ManifestConfirmList',
    mixins: [JeecgListMixin],
    components: {
      JInput,
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        startValue: null,
        endValue: null,
        endOpen: false,
        queryParam:{orgCode:Vue.ls.get('Login_Userinfo').orgCode},
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
            title: '船名',
            align: 'center',
            dataIndex: 'shipName'
          },
          {
            title: '航次 ',
            align: 'center',
            dataIndex: 'voyage'
          },
          {
            title: '发送方代码',
            align: 'center',
            dataIndex: 'senderCode'
          },
          {
            title: '接收方代码',
            align: 'center',
            dataIndex: 'receiverCode'
          },
          {
            title: '确认时间',
            align: 'center',
            dataIndex: 'createTime'
          }
        ],
        ipagination:{
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['20'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: '/manifest/confirm/getConfirmInfo',
        }
      }
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
        this.queryParam = {orgCode:Vue.ls.get('Login_Userinfo').orgCode}
        this.loadData(this.queryParam)
      },


    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>