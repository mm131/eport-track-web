<template>
  <div class="page-header-index-wide">
    <!--<a-row :gutter="24">
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="当日发送南京预配舱单数" :total="dashData.todayNjManifest | NumberFormat">
          <a-tooltip title="企业当日发送南京预配舱单数" slot="action">
            <a-icon type="info-circle-o"/>
          </a-tooltip>
          <div>
          </div>
          <template slot="footer">累计发送数量: <span style="font-weight: bold">{{dashData.totalNjManifest | NumberFormat}}</span></template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="当日发送上海预配舱单数" :total="dashData.todayShManifest | NumberFormat">
          <a-tooltip title="企业当日发送上海预配舱单数" slot="action">
            <a-icon type="info-circle-o"/>
          </a-tooltip>
          <div>
          </div>
          <template slot="footer">累计发送数量: <span style="font-weight: bold">{{dashData.totalShManifest | NumberFormat}}</span></template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="当日追踪提单数" :total="dashData.todayBillCount | NumberFormat">
          <a-tooltip title="企业当日追踪的提单数" slot="action">
            <a-icon type="info-circle-o"/>
          </a-tooltip>
          <div>
          </div>
          <template slot="footer">累计追踪数量: <span style="font-weight: bold">{{dashData.totalBillCount | NumberFormat}}</span></template>
        </chart-card>
      </a-col>
      <a-col :sm="24" :md="12" :xl="6" :style="{ marginBottom: '24px' }">
        <chart-card :loading="loading" title="当日追踪箱数" :total="dashData.todayBoxCount | NumberFormat">
          <a-tooltip title="企业当日追踪的箱数" slot="action">
            <a-icon type="info-circle-o"/>
          </a-tooltip>
          <div>
          </div>
          <template slot="footer">累计追踪数量: <span style="font-weight: bold">{{dashData.totalBoxCount | NumberFormat}}</span></template>
        </chart-card>
      </a-col>
    </a-row>-->

    <!--<a-card :loading="loading" :bordered="false" :body-style="{padding: '0'}">
      <div class="salesCard">
        <a-tabs default-active-key="1" size="large" :tab-bar-style="{marginBottom: '24px', paddingLeft: '16px'}">
          <a-tab-pane loading="true" tab="提单追踪" key="1">
            <a-row>
              <a-col :xl="20" :lg="12" :md="12" :sm="24" :xs="24">
                <bar title="" style="padding-top: 10px;" :dataSource="billStatistics"/>
              </a-col>
            </a-row>
          </a-tab-pane>
          <a-tab-pane tab="箱追踪" key="2">
            <a-row>
              <a-col :xl="20" :lg="12" :md="12" :sm="24" :xs="24">
                <bar title="" style="padding-top: 10px;" :dataSource="boxStatistics"/>
              </a-col>
            </a-row>
          </a-tab-pane>
        </a-tabs>
      </div>
    </a-card>-->
  </div>
</template>

<script>
  import ChartCard from '@/components/ChartCard'
  import ACol from 'ant-design-vue/es/grid/Col'
  import ATooltip from 'ant-design-vue/es/tooltip/Tooltip'
  import Bar from '@/components/chart/Bar'
  import LineChartMultid from '@/components/chart/LineChartMultid'
  import HeadInfo from '@/components/tools/HeadInfo.vue'
  import { getDashboardInfo } from '@/api/api'

  export default {
    name: 'Analysis',
    components: {
      ATooltip,
      ACol,
      ChartCard,
      Bar,
      LineChartMultid,
      HeadInfo
    },
    data() {
      return {
        loading: true,
        center: null,
        billStatistics:[],
        boxStatistics:[],
        dashData: {
          totalBillCount: 0,
          totalBoxCount: 0,
          todayBillCount: 0,
          todayBoxCount: 0,
          totalNjManifest: 0,
          totalShManifest: 0,
          todayNjManifest: 0,
          todayShManifest: 0
        }
      }
    },
    created() {
      setTimeout(() => {
        this.loading = !this.loading
      }, 1000)
      // this.initDashboard()
    },
    methods: {
      initDashboard() {
        let param = Object.assign({});
        getDashboardInfo(param).then((res)=>{
          if(res.success){
            this.dashData = res.result;
            this.billStatistics = res.result.billStatistics;
            this.boxStatistics = res.result.boxStatistics;
          }
        })
      }
    }
  }
</script>

<style lang="scss" scoped>
  .circle-cust {
    position: relative;
    top: 28px;
    left: -100%;
  }

  .extra-wrapper {
    line-height: 55px;
    padding-right: 24px;

    .extra-item {
      display: inline-block;
      margin-right: 24px;

      a {
        margin-left: 24px;
      }
    }
  }

  .statistic {
    padding: 0px !important;
    margin-top: 50px;
  }

  /* 首页访问量统计 */
  .head-info {
    position: relative;
    text-align: left;
    padding: 0 32px 0 0;
    min-width: 125px;

    &.center {
      text-align: center;
      padding: 0 32px;
    }

    span {
      color: rgba(0, 0, 0, .45);
      display: inline-block;
      font-size: .95rem;
      line-height: 42px;
      margin-bottom: 4px;
    }

    p {
      line-height: 42px;
      margin: 0;

      a {
        font-weight: 600;
        font-size: 1rem;
      }
    }
  }
</style>