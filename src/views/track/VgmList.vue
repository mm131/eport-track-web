<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" >
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="舱单状态">
              <a-select v-model="queryParam.status" placeholder="未发送">
                <a-select-option value=0>未发送</a-select-option>
                <a-select-option value=1>发送成功</a-select-option>
                <a-select-option value=2>发送失败</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="船公司">
              <a-select v-model="queryParam.shipCompany" placeholder="全部">
                <a-select-option value="达飞轮船(CMA)">达飞轮船(CMA)</a-select-option>
                <a-select-option value="高丽海运(KMTC)">高丽海运(KMTC)</a-select-option>
                <a-select-option value="正利航业(CNC)">正利航业(CNC)</a-select-option>
                <a-select-option value="澳航海运(ANL)">澳航海运(ANL)</a-select-option>
                <a-select-option value="俄远东(FESCO)">俄远东(FESCO)</a-select-option>
              </a-select>
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
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-upload name="file" :showUploadList="false" :multiple="false"
                        :headers="tokenHeader" :action="importUrl" @change="handleImport" style="margin-left: 8px">
                      <a-button type="primary" icon="import">上传VGM</a-button>
              </a-upload>
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
          <a href="javascript:;"  @click="viewInfo(record)">详情</a>
           <a-divider type="vertical"/>
          <a href="javascript:;"  @click="download(record)">下载</a>
          <a-divider type="vertical"/>
          <a href="javascript:;"  @click="sendVgm(record)">发送</a>
        </span>
      </a-table>
    </div>

    <vgm-modal ref="VgmModal" @ok="modalFormOk"></vgm-modal>
    <download-modal ref="DownloadModal"></download-modal>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import VgmModal from './modules/VgmModal'
  import DownloadModal from './modules/DownloadModal'
  import { getAction } from '@/api/manage'
  import moment from 'moment'
  Vue.prototype.$moment = moment;

  export default {
    name: 'VgmList',
    mixins: [JeecgListMixin],
    components: {
      DownloadModal,
      JInput,
      VgmModal,
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        queryParam: {},
        startValue: null,
        endValue: null,
        endOpen: false,
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
            title: '名称',
            align: 'center',
            dataIndex: 'fileName'
          },
          {
            title: '船公司',
            align: 'center',
            dataIndex: 'shipCompany'
          },
          {
            title: '接收方名称',
            align: 'center',
            dataIndex: 'receiverName'
          },
          {
            title: '接收方代码 ',
            align: 'center',
            dataIndex: 'receiverCode'
          },
          {
            title: '入库时间',
            align: 'center',
            dataIndex: 'createTime'
          },
          {
            title: '发送状态',
            align: 'center',
            dataIndex: 'status'
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
          list: '/vgm/getVgmFile',
          importUrl: 'vgm/importVGM',
          downloadUrl: 'vgm/download'
        }
      }
    },

    computed: {
      importUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}`
      }
    },

    methods: {

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
          this.$message.error('请选择结束时间')
          return;
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

      viewInfo(record){
        this.$refs.VgmModal.loadVgmInfo(record)
      },

      download(record){
          // let url=`${window._CONFIG['domianURL']}/${this.url.downloadUrl}?id=${record.id}`;
          // console.log(url);
          // window.location.href=url;
        this.$refs.DownloadModal.showPage(record.id)

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
      },

      sendVgm(record){
        var that = this;
        this.$confirm({
          title: '提示',
          content: '确认发送这条VGM吗？',
          onOk() {
            getAction(`/vgm/sendVgm`, {
              id:record.id,
            }).then(res => {
              if (res.success) {
                that.$message.success("发送成功");
              }else{
                that.$message.error("发送失败");
              }
              that.loadData(1);
            })
          },
          onCancel() {},
        });
      }


    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>