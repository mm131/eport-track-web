<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" >
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="船代">
              <a-select v-model="queryParam.orgCode">
                <a-select-option value=0>南京甬宁国际船舶代理有限公司</a-select-option>
                <a-select-option value=1>江苏奥远国际船舶代理有限公司南京分公司</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="驳船名称">
              <a-input placeholder="" v-model="queryParam.shipName"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="12">
            <a-form-item label="航次">
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
              <a-button type="primary" @click="importFile" icon="import" style="margin-left: 8px">上传出运表</a-button>
              <a-button type="primary" :disabled="!hasSelected" @click="batchDownload" icon="export" style="margin-left: 8px">批量下载</a-button>
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
        :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange">

        <template
          v-for="col in ['fileName', 'shipName', 'shipId', 'voyageNo']"
          :slot="col"
          slot-scope="text, record, index"
        >
          <div :key="col">
            <a-input
              v-if="record.editable"
              style="margin: -5px 0"
              :value="text"
              @change="e => handleChange(e.target.value, record.id, col)"
            />
            <template v-else>
              {{ text }}
            </template>
          </div>
        </template>

        <template slot="action" slot-scope="text, record, index">
          <div>
            <span v-if="record.editable">
              <a @click="() => save(record.id)">保存</a>
              <a-divider type="vertical" />
              <a-popconfirm title="确定取消吗?" @confirm="() => cancel()">
                <a>取消</a>
              </a-popconfirm>
            </span>
            <span v-else>
              <a :disabled="editingKey !== ''" @click="() => edit(record.id)">修改</a>
              <span v-if="record.modifyNo > 0">
                <a-divider type="vertical" />
                <a href="javascript:;"  @click="modifyRecord(record)">修改记录</a>
              </span>
            </span>

            <span>
              <a-divider type="vertical"/>
              <a href="javascript:;"  @click="viewInfo(record)">详情</a>
              <a-divider type="vertical"/>
              <a href="javascript:;"  @click="download(record)">下载</a>
              <a-divider type="vertical"/>
               <a-dropdown>
                <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
                <a-menu slot="overlay">
                  <a-menu-item>
                     <a href="javascript:;"  @click="forward(record)">转发出运表</a>
                  </a-menu-item>
                  <a-menu-item>
                   <a href="javascript:;"  @click="sendShipmentForm(record)">发送预配舱单</a>
                  </a-menu-item>
                </a-menu>
               </a-dropdown>
            </span>
          </div>
        </template>


      </a-table>
    </div>

    <shipment-form-modal ref="ShipmentFormModal"></shipment-form-modal>
    <shipment-form-upload-modal  ref="ShipmentFormUploadModal" @ok="modalFormOk"></shipment-form-upload-modal>
    <shipment-form-change-rights-modal ref="ShipmentFormChangeRightsModal" @ok2="modalFormOk2"></shipment-form-change-rights-modal>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { getAction,postAction } from '@/api/manage'
  import moment from 'moment'
  import ShipmentFormModal from './modules/ShipmentFormModal'
  import ShipmentFormUploadModal from './modules/ShipmentFormUploadModal'
  import ShipmentFormChangeRightsModal from './modules/ShipmentFormChangeRightsModal'
  Vue.prototype.$moment = moment;

  export default {
    name: 'ShipmentFormList',
    mixins: [JeecgListMixin],
    components: {
      ShipmentFormChangeRightsModal,
      ShipmentFormUploadModal,
      ShipmentFormModal,
      JInput,
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        queryParam: {},
        startValue: null,
        endValue: null,
        endOpen: false,
        id:'',
        selectedRowKeys: [],
        hasSelected: false,
        editingKey: '',
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
            title: '驳船名称',
            align: 'center',
            dataIndex: 'shipName',
            scopedSlots: { customRender: 'shipName' }
          },
          {
            title: '驳船代码',
            align: 'center',
            dataIndex: 'shipId',
            scopedSlots: { customRender: 'shipId' }
          },
          {
            title: '航次 ',
            align: 'center',
            dataIndex: 'voyageNo',
            scopedSlots: { customRender: 'voyageNo' }
          },
          // {
          //   title: '发送状态',
          //   align: 'center',
          //   dataIndex: 'status'
          // },
          {
            title: '创建人',
            align: 'center',
            dataIndex: 'createBy'
          },
          {
            title: '创建时间',
            align: 'center',
            dataIndex: 'createTime'
          },
          {
            title: '修改人',
            align: 'center',
            dataIndex: 'updateBy'
          },
          {
            title: '修改时间',
            align: 'center',
            dataIndex: 'updateTime'
          },
          {
            title: '操作',
            align: 'center',
            dataIndex: 'action',
            fixed:'right',
            width: 200,
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
          list: '/shipmentForm/getShipmentFormFile',
          importUrl: 'shipmentForm/importShipmentForm',
          downloadUrl: 'shipmentForm/download'
        }
      }
    },

    computed: {
      importUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}`
      }
    },

    methods: {

      modalFormOk2(){
        this.loadData(1);
      },

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


      batchDownload(){
        var checkInfo=[];
        var ids='';
        for(var i=0;i<this.selectedRowKeys.length;i++){
          var checkId=this.selectedRowKeys[i];
          if(ids.length>0){
            ids=ids+','+checkId;
          }else{
            ids=ids+checkId;
          }

          var shipmentForm=this.dataSource.filter(item => item.id === checkId)[0];
          var shipFlag=shipmentForm.shipName+shipmentForm.voyageNo;
          if(!checkInfo.includes(shipFlag)){
            checkInfo.push(shipFlag);
          }
        }
        if(checkInfo.length>1){
          this.$message.error("所选记录的驳船名称、航次必须相同");
        }else{
          let url=`${window._CONFIG['domianURL']}/${this.url.downloadUrl}?id=${ids}`;
          window.location.href=url;
          this.selectedRowKeys=[];
          this.hasSelected=false;
        }

      },

      onSelectChange(selectedRowKeys) {
        this.selectedRowKeys = selectedRowKeys;
        if(this.selectedRowKeys.length>0){
          this.hasSelected=true;
        }else{
          this.hasSelected=false;
        }
      },

      viewInfo(record){
        this.$refs.ShipmentFormModal.loadShipmentFormInfo(record)
      },

      download(record){
        let url=`${window._CONFIG['domianURL']}/${this.url.downloadUrl}?id=${record.id}`;
        window.location.href=url;
      },


      handleChange(value, id, column) {
        const newData = [...this.dataSource];
        const target = newData.filter(item => id === item.id)[0];
        if (target) {
          target[column] = value;
          this.dataSource = newData;
        }
      },
      edit(id) {
        const newData = [...this.dataSource];
        const target = newData.filter(item => id === item.id)[0];
        this.editingKey = id;
        if (target) {
          target.editable = true;
          this.dataSource = newData;
        }
      },

      save(id) {
        const newData = [...this.dataSource];
        const target = newData.filter(item => id === item.id)[0];
        if (target) {
          postAction(`/shipmentForm/updateShipmentFormFile`,target).then(res=>{
            if(res.success){
              delete target.editable;
              this.dataSource = newData;
              this.loadData();
              this.$message.success("修改成功");
            }else{
              this.loadData();
              this.$message.error("修改失败");
            }
            this.editingKey = '';
          })
        }

      },

      cancel() {
        this.editingKey = '';
        this.loadData();
      },


      importFile(){
        this.$refs.ShipmentFormUploadModal.loadUploadPage()
      },

      forward(record){
        this.$refs.ShipmentFormChangeRightsModal.changeRigths(record)
      },

      sendShipmentForm(record){
        var that = this;
        this.$confirm({
          title: '提示',
          content: '确认发送这条出运表吗？',
          onOk() {
            getAction(`/shipmentForm/sendShipmentForm`, {
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