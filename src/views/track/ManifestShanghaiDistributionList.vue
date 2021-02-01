<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" >
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="提运单号">
              <a-input placeholder="" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="箱号">
              <a-input placeholder="" v-model="queryParam.boxNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="干线船名">
              <a-input placeholder="" v-model="queryParam.lineName"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="干线航次">
              <a-input placeholder="" v-model="queryParam.lineVoyage"></a-input>
            </a-form-item>
          </a-col>


          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-upload name="file" :showUploadList="false" :multiple="false"
                        :headers="tokenHeader" :action="importUrl1" @change="handleImport" style="margin-left: 8px">
                      <a-button type="primary" icon="import">上传FA清单</a-button>
              </a-upload>
               <a-upload name="file" :showUploadList="false" :multiple="false"
                         :headers="tokenHeader" :action="importUrl2" @change="handleImport" style="margin-left: 8px">
                      <a-button type="primary" icon="import">上传预录入表</a-button>
              </a-upload>

              <a-button type="primary" :disabled="!hasSelected" @click="exportFaFileBefore" icon="export" style="margin-left: 8px">通关前改配</a-button>
              <a-button type="primary" :disabled="!hasSelected" @click="exportFaFileAfter" icon="export" style="margin-left: 8px">通关后改配</a-button>
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
        :row-selection="{ selectedRowKeys: selectedRowKeys,selectedRows:selectedRows, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a href="javascript:;"  @click="exportFile(record)">通关后改配</a>
        </span>
      </a-table>
    </div>

    <manifest-shanghai-distribution-before-modal ref="ManifestShanghaiDistributionBeforeModal" @ok="modalFormOk"></manifest-shanghai-distribution-before-modal>
    <manifest-shanghai-distribution-after-modal ref="ManifestShanghaiDistributionAfterModal" @ok="modalFormOk"></manifest-shanghai-distribution-after-modal>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import moment from 'moment'
  import ManifestShanghaiDistributionAfterModal from './modules/ManifestShanghaiDistributionAfterModal'
  import ManifestShanghaiDistributionBeforeModal from './modules/ManifestShanghaiDistributionBeforeModal'

  Vue.prototype.$moment = moment;

  export default {
    name: 'ManifestShanghaiDistributionList',
    mixins: [JeecgListMixin],
    components: {
      ManifestShanghaiDistributionBeforeModal,
      ManifestShanghaiDistributionAfterModal,
      JInput,
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        queryParam: {},
        selectedRowKeys: [],
        selectedRows:[],
        hasSelected: false,
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
            title: '提运单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '报关单号',
            align: 'center',
            dataIndex: 'customsDeclarationNumber'
          },
          {
            title: '预录入号',
            align: 'center',
            dataIndex: 'preEntryNumber'
          },
          {
            title: '干线船名',
            align: 'center',
            dataIndex: 'lineName'
          },
          {
            title: '干线航次',
            align: 'center',
            dataIndex: 'lineVoyage'
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
          list: '/distribution/getFaList',
          importUrl: 'distribution/importFA'
        }
      }
    },

    computed: {
      importUrl1: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}?uploadType=1`
      },
      importUrl2: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}?uploadType=2`
      }
    },

    methods: {

      onSelectChange(selectedRowKeys,selectedRows) {
        this.selectedRowKeys = selectedRowKeys;
        this.selectedRows = selectedRows;
        if(this.selectedRowKeys.length>0){
          this.hasSelected=true;
        }else{
          this.hasSelected=false;
        }
      },

      searchQuery(){
        this.loadData(this.queryParam);
      },


      //通关前改配
      exportFaFileBefore(){
        var checkInfo=[];
        for(var i=0;i<this.selectedRows.length;i++){
          var preEntryNumber=this.selectedRows[i].preEntryNumber;
          if(!checkInfo.includes(preEntryNumber)){
            checkInfo.push(preEntryNumber);
          }
        }
        if(checkInfo.length>1){
          this.$message.error('您勾选的提单具有多个不同的预录入号，请重新勾选！');
          return
        }else{
          this.$refs.ManifestShanghaiDistributionBeforeModal.loadBillInfo(this.selectedRows)
        }
      },


      //单票通关后改配
      exportFile(record){
        this.$refs.ManifestShanghaiDistributionAfterModal.loadContaInfo(record,1);
      },

      //通关后改配，勾选了多条记录，根据预录入号判断是单票还是多票
      exportFaFileAfter(){
        var checkInfo=[];
        for(var i=0;i<this.selectedRows.length;i++){
          var preEntryNumber=this.selectedRows[i].preEntryNumber;
          if(!checkInfo.includes(preEntryNumber)){
            checkInfo.push(preEntryNumber);
          }
        }
        if(checkInfo.length>1){
          this.$refs.ManifestShanghaiDistributionAfterModal.loadContaInfo(this.selectedRows,2)
        }else{
          this.$refs.ManifestShanghaiDistributionAfterModal.loadContaInfo(this.selectedRows,3)
        }

      },

      handleImport(info) {
        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (info.file.status === 'done') {
          if (info.file.response.success) {
            if (info.file.response.code === 200) {
              this.$message.success(info.file.response.message || `${info.file.name} 文件上传成功`)
              this.loadData(1)
            } else {
              this.$message.error('上传失败')
            }
          } else {
            this.$message.error(`${info.file.response.message}`)
          }
        } else if (info.file.status === 'error') {
          this.$message.error(`文件上传失败: ${info.file.msg} `)
        }
      }


    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>