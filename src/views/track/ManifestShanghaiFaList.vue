<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" >
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="驳船船名航次">
              <a-input placeholder="" v-model="queryParam.shipNameVoyage"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset1" icon="reload" style="margin-left: 8px">重置</a-button>
                  <a-button type="primary" @click="handleImport" icon="import" style="margin-left: 8px">上传报文</a-button>
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
          <a href="javascript:;" @click="refreshFaContent(record)">刷新</a>
           <a-divider type="vertical"/>
          <a href="javascript:;" @click="getFaContent(record)">详情</a>
        </span>

        <span slot="billNoCountAndBoxNoCount" slot-scope="text, record">
            {{record.billCount+ '个/' + record.boxCount + '箱'}}
        </span>

        <span slot="unSendBillCount" slot-scope="text, record">
          <span v-bind:style="{ color: record.unSendBillCount > 0 ? 'red' : 'black' }">
                {{record.unSendBillCount}}
          </span>个
        </span>

        //未发送
        <span slot="unSendBillCount" slot-scope="text, record">
          <span v-bind:style="{ color: record.unSendBillCount > 0 ? 'red' : 'black' }">
                {{record.unSendBillCount}}
          </span>箱
        </span>

        //未运抵
        <span slot="unArrivalBoxCount" slot-scope="text, record">
          <span v-bind:style="{ color: record.unArrivalBoxCount > 0 ? 'red' : 'black' }">
                {{record.unArrivalBoxCount}}
          </span>箱
        </span>

        //海关未放行
        <span slot="unCustomReleaseBoxCount" slot-scope="text, record">
          <span v-bind:style="{ color: record.unCustomReleaseBoxCount > 0 ? 'red' : 'black' }">
                {{record.unCustomReleaseBoxCount}}
          </span>箱
        </span>

        //码头未放行
        <span slot="unPortReleaseBoxCount" slot-scope="text, record">
          <span v-bind:style="{ color: record.unPortReleaseBoxCount > 0 ? 'red' : 'black' }">
                {{record.unPortReleaseBoxCount}}
          </span>箱
        </span>

        //码头未配载
        <span slot="unStowageBoxCount" slot-scope="text, record">
          <span v-bind:style="{ color: record.unStowageBoxCount > 0 ? 'red' : 'black' }">
                {{record.unStowageBoxCount}}
          </span>箱
        </span>

      </a-table>
    </div>

    <manifest-shanghai-fa-modal ref="ManifestShanghaiFaModal"></manifest-shanghai-fa-modal>
    <manifest-shanghai-fa-upload-modal ref="ManifestShanghaiFaUploadModal" @ok="modalFormOk"></manifest-shanghai-fa-upload-modal>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import moment from 'moment'
  import ManifestShanghaiFaModal from './modules/ManifestShanghaiFaModal'
  import ManifestShanghaiFaUploadModal from './modules/ManifestShanghaiFaUploadModal'
  import { postAction,getAction } from '@/api/manage'

  export default {
    name: 'ManifestShanghaiFaList',
    mixins: [JeecgListMixin],
    components: {
      ManifestShanghaiFaUploadModal,
      ManifestShanghaiFaModal,
      JInput,
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
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
            title: '驳船船期',
            align: 'center',
            dataIndex: 'shipPeriod',
          },
          {
            title: '驳船船名航次',
            align: 'center',
            dataIndex: 'shipNameVoyage'
          },
          {
            title: '提单数/箱数',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'billNoCountAndBoxNoCount' }
          },
          {
            title: '舱单未发送提单',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'unSendBillCount' }
          },
          {
            title: '未运抵箱',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'unArrivalBoxCount' }
          },
          {
            title: '海关未放行箱',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'unCustomReleaseBoxCount' }
          },
          {
            title: '码头未放行箱',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'unPortReleaseBoxCount' }
          },
          {
            title: '码头未配载箱',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'unStowageBoxCount' }
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
          list: '/fa/getFaList',
        }
      }
    },


    methods: {

      searchQuery(){
        this.loadData(this.queryParam);
      },

      handleImport(){
        this.$refs.ManifestShanghaiFaUploadModal.showUploadPage();
      },

      searchReset1() {
        this.queryParam = {}
        this.loadData(1);
      },

      getFaContent(record) {
        this.$refs.ManifestShanghaiFaModal.loadFaContaInfo(record);
      },


      modalFormOk(){
        this.loadData(1);
        this.refreshFaAll()
      },

      refreshFaAll(){
        postAction(`/fa/refreshFaAll`, {}
        ).then(res => {
          if(res.success){
            this.loadData(1);
          }
        })
      },



      refreshFaContent(record) {
        if(record.shipNameVoyage==null){
          this.$message.error("驳船船名航次不能为空！");
          return;
        }
        postAction(`/fa/refreshFaByShipNameVoyage`, {
          shipNameVoyage: record.shipNameVoyage
        }).then(res => {
          if(res.success){
            this.$message.success("刷新成功！");
            this.loadData(1);
          }else{
            this.$message.error("刷新失败！");
          }
        })
      },

    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>