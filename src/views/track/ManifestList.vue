<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="舱单状态">
              <a-select v-model="queryParam.status" placeholder="未发送">
                <a-select-option value=0>未发送</a-select-option>
                <a-select-option value=1>发送成功</a-select-option>
                <a-select-option value=2>发送失败</a-select-option>
                <a-select-option value=3>确认成功</a-select-option>
                <a-select-option value=4>确认失败</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="提运单号">
              <a-input placeholder="" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="二程船名">
              <a-input placeholder="" v-model="queryParam.shipName"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="二程航次">
              <a-input placeholder="" v-model="queryParam.voyageNo"></a-input>
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

          <a-col :md="5" :sm="8">
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

          <a-col :md="5" :sm="6">
            <a-form-item label="总署运抵状态">
              <a-switch checked-children="未运抵" un-checked-children="全部" default-checked @change="onChange"/>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset1" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-button type="primary" @click="handleImport" icon="import" style="margin-left: 8px">上传报文</a-button>
              <a-button type="primary" :disabled="!hasSelected" @click="batchConfirm"  style="margin-left: 8px">批量确认</a-button>
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
        @expand="expand"
        :expandedRowKeys="expandedRowKeys"
        :row-selection="{ selectedRowKeys: selectedRowKeys,selectedRows:selectedRows, onChange: onSelectChange}"
        @change="handleTableChange">

         <span slot="customsStatusType" slot-scope="text, record">
            <span v-if="record.customsStatus==0 || record.customsStatus==null">未运抵</span>
            <span v-else>运抵正常</span>
         </span>

        <span slot="planArriveTimeType" slot-scope="text, record">
             <span>{{record.planArriveTime}}</span>
             <span><img  :src=imgUrl  style="width: 30px;height: 30px;"  @click="reFresh(record)"/></span>
        </span>


        <span slot="action" slot-scope="text, record">
          <a href="javascript:;"  @click="viewInfo(record)">校验</a>
          <a-divider type="vertical"/>
          <a href="javascript:;"  @click="sendManifest(record)">发送</a>
          <a-divider type="vertical"/>
          <span  v-if="record.status=='未发送' || record.status=='发送亿通失败' ">
            <a  href="javascript:;"  @click="viewCode(record)" disabled="disabled">确认</a>
          </span>
          <span  v-if="record.status=='发送亿通成功' || record.status=='舱单确认失败' || record.status=='舱单确认成功'">
            <a  href="javascript:;"  @click="viewCode(record)">确认</a>
          </span>
          <a-divider type="vertical"/>
           <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record)">
             <a>删除</a>
           </a-popconfirm>
        </span>

        <a-table
          slot="expandedRowRender"
          slot-scope="text"
          rowKey="boxId"
          :columns="innerColumns"
          :data-source="innerData"
          :pagination="false"
        >
          <span slot="arrivalType" slot-scope="text, record">
            <span v-if="record.arrivalMessageSendTime!=null">Y</span>
            <span v-else>N</span>
          </span>

          <span slot="customType" slot-scope="text, record">
            <span v-if="record.customReleaseTime!=null">Y</span>
            <span v-else>N</span>
          </span>
        </a-table>
      </a-table>
    </div>

    <manifest-modal ref="ManifestModal" @ok="modalFormOk"></manifest-modal>
    <manifest-upload-modal ref="ManifestUploadModal" @ok="modalFormOk"></manifest-upload-modal>
    <manifest-confirm-code  ref="ManifestConfirmCode" @ok="modalFormOk"></manifest-confirm-code>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import ManifestModal from './modules/ManifestModal'
  import { postAction,getAction } from '@/api/manage'
  import moment from 'moment'
  import ManifestConfirmCode from './modules/ManifestConfirmCode'
  import ManifestUploadModal from './modules/ManifestUploadModal'
  Vue.prototype.$moment = moment


  export default {
    name: 'ManifestList',
    mixins: [JeecgListMixin],
    components: {
      ManifestUploadModal,
      ManifestConfirmCode,
      JInput,
      ManifestModal
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        startValue: null,
        endValue: null,
        endOpen: false,
        imgUrl:null,
        selectedRowKeys: [],
        selectedRows:[],
        hasSelected: false,
        queryParam:{orgCode:Vue.ls.get('Login_Userinfo').orgCode, status:null,customsStatus:0},
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
            title: '操作',
            align: 'center',
            dataIndex: 'action',
            width: 200,
            scopedSlots: { customRender: 'action' }
          },
          {
            title: '舱单状态',
            align: 'center',
            dataIndex: 'status'
          },
          {
            title: '总署运抵状态',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'customsStatusType' }
          },
          {
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '二程船名',
            align: 'center',
            dataIndex: 'shipName'
          },
          {
            title: '二程航次 ',
            align: 'center',
            dataIndex: 'voyage'
          },
          {
            title: '驳船船名航次 ',
            align: 'center',
            dataIndex: 'shipNameVoyage'
          },
          {
            title: '接收方名称(接收方代码)',
            align: 'center',
            dataIndex: 'receiverCode'
          },
          {
            title: '计划靠泊时间',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'planArriveTimeType' }
          },
          {
            title: '实际靠泊时间',
            align: 'center',
            dataIndex: 'factArriveTime'

          },
          {
            title: '发送方式',
            align: 'center',
            dataIndex: 'sendMethod'
          },
          {
            title: '计划发送时间',
            align: 'center',
            dataIndex: 'sendTime'
          },
          {
            title: '更新时间',
            align: 'center',
            dataIndex: 'modifyTime'
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
          list: '/manifest/getManifest',
        },

        expandedRowKeys: [],
        innerData:[],
        innerColumns:[
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'boxNo'
          },
          {
            title: '运抵',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'arrivalType' }
          },
          {
            title: '海放',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'customType' }
          },
          {
            title: '码放',
            align: 'center',
            dataIndex: 'portReleaseType'
          },
          {
            title: '箱型',
            align: 'center',
            dataIndex: 'boxSize'
          },
          {
            title: '持箱人',
            align: 'center',
            dataIndex: 'boxOperator'
          },
          {
            title: '箱状态',
            align: 'center',
            dataIndex: 'boxStatus'
          },
          {
            title: '进场方式',
            align: 'center',
            dataIndex: 'arrivalSpaceType'
          }
        ],

      }
    },


    mounted(){
      var date1=new Date().toLocaleDateString()+' 00:00:00';
      var date2=new Date().toLocaleDateString()+' 23:59:59';
      this.startValue=moment(date1,'YYYY-MM-DD HH:mm:ss');
      this.endValue=moment(date2,'YYYY-MM-DD HH:mm:ss');

      this.imgUrl='/刷新.png';
    },

    methods: {

      onSelectChange(selectedRowKeys,selectedRows) {
        this.selectedRowKeys = selectedRowKeys;
        this.selectedRows = selectedRows;
        console.log(this.selectedRows);
        if(this.selectedRowKeys.length>0){
          this.hasSelected=true;
        }else{
          this.hasSelected=false;
        }
      },

      viewInfo(record){
        this.$refs.ManifestModal.loadShipInfo(record)
        this.$refs.ManifestModal.title = '舱单修改前后对照'
      },

      sendManifest(record){
        var that = this;
        this.$confirm({
          title: '提示',
          content: '确认发送这条舱单吗？',
          onOk() {
            postAction(`/manifest/sendManifest`, {
              id:record.id,
              manifestId:record.manifestId,
              afterModify:record.afterModify
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
      },

      expand (expanded, record) {
        this.expandedRowKeys=[];
        this.innerData=[];
        if (expanded) {
          this.expandedRowKeys = [record.id];
          getAction(`/manifest/getManifestConta`,{billNo:record.billNo}).then(res=>{
            if(res.success){
              this.innerData=res.result;
            }else{
              this.$message.error("箱货信息获取失败")
            }
          });
        }
      },


      onChange(checked) {
        if(checked===true){
          this.queryParam.customsStatus=0
          this.loadData(1)
        }else{
          this.queryParam.customsStatus=1
          this.loadData(1)
        }
      },


      handleDelete(record){
        console.log(record.status);
        if(record.status!=="未发送" && record.status!=="发送亿通失败"){
          this.$message.error('该舱单已发送亿通成功，不可删除！')
          return;
        }
        console.log(record.manifestId);
        postAction(`/manifest/deleteManifest`,{
          manifestId: record.manifestId,
        }).then(res => {
          if(res.success){
            this.$message.success("删除成功");
            this.loadData(this.queryParam);
          }else{
            this.$message.error("删除失败");
          }
        })
      },


      handleImport(){
        this.$refs.ManifestUploadModal.showUploadPage()
      },

      viewCode(record){
        this.$refs.ManifestConfirmCode.showConfirmCode(record,1)
      },


      batchConfirm(){
        var checkInfo=[];
        for(var i=0;i<this.selectedRows.length;i++){
          console.log("lallalalalala");
          console.log(this.selectedRows[i]);
          var receiverCode=this.selectedRows[i].receiverCode;
          if(!checkInfo.includes(receiverCode)){
            checkInfo.push(receiverCode);
          }
        }
        if(checkInfo.length>1){
          this.$message.error('批量确认时，接收方代码必须一致，请重新勾选！');
          return
        }else{
          this.$refs.ManifestConfirmCode.showConfirmCode(this.selectedRows,2)
        }


      },


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
        console.log(this.queryParam.endDate);
        console.log(this.queryParam.startDate);
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

      reFresh(record){
        postAction(`/manifest/updateArriveTime`,{
          shipName: record.shipName,
          manifestId: record.manifestId
        }).then(res => {
          if(res.success){
            this.$message.success("更新成功");
            this.loadData(this.queryParam);
          }else{
            this.$message.error("更新失败");
          }
        })
      }


    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>