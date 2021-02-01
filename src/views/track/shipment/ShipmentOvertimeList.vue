<template>
  <a-card :bordered="false">

    <!--查询-->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="箱号">
              <a-input placeholder="输入箱号查询" v-model="queryParam.boxNo"></a-input>
            </a-form-item>
          </a-col>

          <!--          <a-col :md="4" :sm="13">
                      <a-form-item label="是否超时">
                        <a-switch checked-children="超时" un-checked-children="未超时" default-checked @change="onChange"/>
                      </a-form-item>
                    </a-col>-->

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>

        </a-row>

        <!-- <div>查询到0个货箱</div>-->

      </a-form>
    </div>

    <div class="table-operator">

      <a-button @click="downloadTemplate" type="primary" icon="download">模板</a-button>

      <!--<a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel" style="margin-left: 8px">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>-->

      <a-button @click="handleContainerImport()" type="primary" icon="import" style="margin-left: 8px">导入</a-button>

    </div>

    <a-table
      ref="table"
      size="middle"
      bordered
      rowKey="rowIndex"
      :columns="columns"
      :dataSource="dataSource"
      :pagination="ipagination"
      :loading="loading"
      @change="handleTableChange">


      <!--南京港运抵未装船箱数-->
      <span slot="out" slot-scope="text, record">
          <a href="javascript:;"
             v-bind:style="{ color: parseInt(record.wetherOutTime) >0 ? 'red' : 'black' }">
            {{record.wetherOutTime}}天</a>
      </span>


    </a-table>

    <!--表单区域-->
    <container-import-modal ref="containerImportModal" @close="importModalOk"
                            @ok="importModalOk"></container-import-modal>

  </a-card>
</template>

<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { filterObj } from '@/utils/util'
  import { getAction } from '@/api/manage'
  import Vue from 'vue'
  import containerImportModal from './modules/ContainerImportModal'

  export default {
    name: 'ShipmentOvertimeList',
    mixins: [JeecgListMixin],
    components: {
      containerImportModal
    },
    data() {
      return {
        description: '装船超时管理页面',
        isorter: { column: 'create_time' },
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
            width: 100,
            dataIndex: 'boxNo'
          },
          {
            title: '箱型',
            align: 'center',
            customRender: function(text, r, index) {
              console.log(text)
              return (text.boxSize + text.boxType)
            }
          },
          /*{
                      dataIndex: 'boxType'
            title: '出门时间',
            align: 'center',
            dataIndex: 'njDepartureTime'
          },*/
          {
            title: '进场时间',
            align: 'center',
            dataIndex: 'njEntryTime'
          },
          {
            title: '装船时间',
            align: 'center',
            dataIndex: 'njLoadTime'
          },
          {
            title: '超期天数',
            align: 'center',
            dataIndex: 'wetherOutTime',
            scopedSlots: { customRender: 'out' }
          }

        ],
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30', '50'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        url: {
          list: '/shipment/task/list',
          templateUrl: 'shipment/task/template',
          importExcelUrl: 'shipment/task/importExcel'
        }
      }
    },
    methods: {
      onChange(checked) {
        if (checked === true) {
          this.queryParam.outTimeFlag = '1'
          this.loadData(1)
        } else {
          this.queryParam.outTimeFlag = '0'
          this.loadData(1)
        }
      },
      searchQuery() {
        //从本地仓库获取 orgCode
        this.queryParam.orgCode = Vue.ls.get('Login_Userinfo').orgCode
        this.loadData(1)
      },
      searchReset() {
        this.queryParam = {}
        this.queryParam.orgCode = Vue.ls.get('Login_Userinfo').orgCode
        this.loadData(1)
      },
      loadData(arg) {
        if (!this.url.list) {
          this.$message.error('请设置url.list属性!')
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1
        }
        var params = this.getQueryParams()//查询条件
        console.log('查询参数：{}', params)
        this.loading = true
        getAction(this.url.list, params).then((res) => {
          console.log('返回结果:{}', res)
          if (res.success) {
            this.dataSource = res.result.records
            this.ipagination.total = res.result.total
          }
          if (res.code === 510) {
            this.$message.warning(res.message)
          }
          this.loading = false
        })
      },
      getQueryParams() {
        //获取查询条件
        let sqp = {}
        if (this.superQueryParams) {
          sqp['superQueryParams'] = encodeURI(this.superQueryParams)
        }
        var param = Object.assign(sqp, this.queryParam, this.isorter, this.filters)
        param.field = this.getQueryField()
        param.pageNo = this.ipagination.current
        param.pageSize = this.ipagination.pageSize
        return filterObj(param)
      },
      downloadTemplate: function() {
        let url = `${window._CONFIG['domianURL']}/${this.url.templateUrl}`
        window.location.href = url
      },
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      },
      //导入
      handleContainerImport() {
        this.$refs.containerImportModal.show()
        this.$refs.containerImportModal.cleanCode()
        this.$refs.containerImportModal.getCaptchaImg()
      },
      importModalOk() {
        this.searchQuery()
      }
    }
  }
</script>

<style scoped>

</style>