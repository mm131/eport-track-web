<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" >
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="运输工具编号">
              <a-input placeholder="" v-model="queryParam.shipId"></a-input>
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

          <a-col :md="6" :sm="12">
            <a-form-item label="放行状态">
              <a-switch checked-children="未放行" un-checked-children="全部" default-checked @change="onChange"/>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset1" icon="reload" style="margin-left: 8px">重置</a-button>
              <a-button type="primary" @click="addManifest"  style="margin-left: 8px">新增</a-button>

            </span>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importUrl" @change="handleImport" style="margin-left: 8px">
                <a-button type="primary" icon="import">上传报文</a-button>
              </a-upload>
              <a-button @click="handleNjInfo()" type="primary" icon="reload" style="margin-left: 8px">获取运抵报告状态</a-button>
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
          <a href="javascript:;" @click="getManifestConta(record)">详情</a>
          <a-divider type="vertical"/>
          <a href="javascript:;" @click="viewInfo(record)">查看报文</a>
          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">

              <a-menu-item>
                  <a href="javascript:;" @click="editManifest(record)">编辑</a>
              </a-menu-item>

              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>

            </a-menu>
          </a-dropdown>
        </span>

        <span slot="checkFlagStyle" slot-scope="text, record">
          <span v-if="record.checkFlag==='Y' && record.deleteGuid ===null" style="color:darkgreen">通过(Y)</span>
          <span v-else-if="record.checkFlag==='N' && record.deleteGuid ===null" style="color:red">未通过(N)</span>
          <span v-else-if="record.checkFlag==='1' && record.deleteGuid ===null" >运抵比对成功(1)</span>
          <span v-else-if="record.deleteGuid !=null">删除待审批(010)</span>
          <span v-else>
            待审批
          </span>
        </span>


        <span slot="customsType" slot-scope="text, record">
             <span v-if="record.customsCode !=null">{{record.abbrCustoms}}({{record.customsCode}})</span>
        </span>


        <span slot="lineFlagType" slot-scope="text, record">
          <span v-if="record.lineFlag==='1'">干线(1)</span>
          <span v-else-if="record.lineFlag==='2'">支线(2)</span>
          <span v-else>
            {{record.lineFlag}}
          </span>
        </span>

        <span slot="releaseType" slot-scope="text, record">
          <span v-if="record.releaseStatus===null || record.releaseStatus==='未放行(01)'">无放行信息</span>
          <span v-else style="color:darkgreen">
            {{record.releaseStatus}}
          </span>
        </span>

        <span slot="newCheckFlag" slot-scope="text, record">
          <span v-if="record.manifestStatus==='01' " style="color:darkgreen;text-decoration:underline;">
             <a-tooltip>
              <template slot="title">
                 {{record.checkNewNotes}}
              </template>
               通过(01)
            </a-tooltip>
          </span>
          <span v-else-if="record.manifestStatus==='03'" style="color:red;text-decoration:underline;">
            <a-tooltip>
              <template slot="title">
                 {{record.checkNewNotes}}
              </template>
               退单(03)
            </a-tooltip>
          </span>
          <span v-else-if="record.manifestStatus==='010'">删除待审批(010)</span>
          <span v-else-if="record.manifestStatus==='05'" style="color:red;text-decoration:underline;">
             <a-tooltip>
              <template slot="title">
                 {{record.checkNewNotes}}
              </template>
               删除退单
            </a-tooltip>
          </span>
          <span v-else>
            待审批
          </span>
        </span>

      </a-table>
    </div>

    <manifest-nanjing-modal ref="ManifestNanjingModal" @ok="modalFormOk"></manifest-nanjing-modal>
    <manifest-nanjing-conta-modal ref="ManifestNanjingContaModal"></manifest-nanjing-conta-modal>
    <manifest-nanjing-info-modal ref="ManifestNanjingInfoModal"></manifest-nanjing-info-modal>
    <manifest-nanjing-info-status-modal @click="handleNjInfo()"
                                        ref="ManifestNanjingInfoStatusModal"></manifest-nanjing-info-status-modal>
    <manifest-nanjing-add-modal ref="ManifestNanjingAddModal"></manifest-nanjing-add-modal>

    <manifest-nanjing-add-conta-type ref="ManifestNanjingAddContaType"></manifest-nanjing-add-conta-type>
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import ManifestNanjingModal from './modules/ManifestNanjingModal'
  import ManifestNanjingContaModal from './modules/ManifestNanjingContaModal'
  import moment from 'moment'
  import { postAction,getAction } from '@/api/manage'
  import ManifestNanjingInfoModal from './modules/ManifestNanjingInfoModal.vue'
  import ManifestNanjingInfoStatusModal from './modules/ManifestNanjingInfoStatusModal.vue'
  import ManifestNanjingAddModal from './modules/ManifestNanjingAddModal'
  import ACol from 'ant-design-vue/es/grid/Col'
  import ManifestNanjingAddContaType from './modules/ManifestNanjingAddContaType'

  Vue.prototype.$moment = moment

  export default {
    name: 'ManifestNanjingList',
    mixins: [JeecgListMixin],
    components: {
      ManifestNanjingAddContaType,
      ACol,
      ManifestNanjingAddModal,
      ManifestNanjingModal,
      ManifestNanjingContaModal,
      JInput,

      ManifestNanjingInfoModal,
      ManifestNanjingInfoStatusModal
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        startValue: null,
        endValue: null,
        endOpen: false,
        obj:[],
        flag: false,
        certificate:[],
        queryParam:{orgCode:Vue.ls.get('Login_Userinfo').orgCode, releaseStatus:'1'},
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
            title: '海关审核(新)',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'newCheckFlag' }
          },
          {
            title: '放行状态',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'releaseType' }
          },
          {
            title: '运抵报告状态',
            align: 'center',
            dataIndex: 'arrivalReportStatus'
          },
          {
            title: '海关代码',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'customsType' }
          },
          {
            title: '运输工具编号',
            align: 'center',
            dataIndex: 'shipId'
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
            title: '件数',
            align: 'center',
            dataIndex: 'packNo'
          },
          {
            title: '毛重(公斤)',
            align: 'center',
            dataIndex: 'grossWeight'
          },
          {
            title: '航线标志',
            align: 'center',
            dataIndex: '',
            scopedSlots: { customRender: 'lineFlagType' }
          },
          {
            title: '入库时间',
            align: 'center',
            dataIndex: 'updateTime'
          },
          {
            title: '审核时间',
            align: 'center',
            dataIndex: 'perTime'
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
          list: '/manifestNanjing/getManifest',
          importUrl: 'manifestNanjing/importManifestNanjing',
          templateUrl: 'download/template'
        }
      }
    },
    mounted() {
      clearInterval(this.timer);
      this.setTimer();
      var date1=new Date().toLocaleDateString()+' 00:00:00';
      var date2=new Date().toLocaleDateString()+' 23:59:59';
      this.startValue=moment(date1,'YYYY-MM-DD HH:mm:ss');
      this.endValue=moment(date2,'YYYY-MM-DD HH:mm:ss');
      let obj = document.getElementsByTagName("object");
      let flag = false;

      this.obj=new NetONEX();
      this.obj.setupObject();

    },
    computed: {
      importUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}`
      }
    },

    methods: {

      setTimer: function() {
        this.timer = setInterval(() => {
          this.loadData(this.queryParam)
        }, 30000)
      },

      moment,

      onChange(checked) {
        if(checked===true){
          this.queryParam.releaseStatus='1'
          this.loadData(1)
        }else{
          this.queryParam.releaseStatus='0'
          this.loadData(1)
        }
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

      viewInfo(record) {
        this.$refs.ManifestNanjingModal.loadShipInfo(record)
        this.$refs.ManifestNanjingModal.title = '南京预配舱单信息'
      },

      getManifestConta(record) {
        this.$refs.ManifestNanjingContaModal.loadManifestContaInfo(record)
      },

      handleImport(info) {
        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (info.file.status === 'done') {
          if (info.file.response.success) {
            if(info.file.response.message==="allRight"){
              var orgCode=Vue.ls.get('Login_Userinfo').orgCode;
              var oldXML=info.file.response.result.oldXml;
              var newXML=info.file.response.result.newXml;
              var oldSignXML="";
              var newSignXml="";
              try{
                getAction(`/manifestNanjing/getWhetherSign`,{orgCode:orgCode}).then(res=>{
                  if(res.success){
                    if(res.result==='0'){//需要加签
                      if(oldXML !== null && oldXML !== ''){
                        oldSignXML = this.getSignedXml(oldXML);
                        if(oldSignXML===undefined || oldSignXML === '' || oldSignXML===null){
                          this.$message.error("检查是否插入USBKEY；或是否安装国信UK驱动")
                          return;
                        }
                      }
                      newSignXml = this.getSignedXml(newXML);
                      if(newSignXml===undefined || newSignXml === '' || newSignXml===null){
                        this.$message.error("检查是否插入USBKEY；或是否安装国信UK驱动")
                        return;
                      }
                      postAction(`/manifestNanjing/sendMenifestToMQ`,{oldXML:oldSignXML,newXML:newSignXml}).then(res=>{
                        if(res.success){
                          this.$message.success("上传成功");
                        }else{
                          this.$message.error("上传失败");
                        }
                      })
                    }else{     //不需要加签
                      postAction(`/manifestNanjing/sendMenifestToMQ`,{oldXML:oldXML,newXML:newXML}).then(res=>{
                        if(res.success){
                          this.$message.success("上传成功");
                        }else{
                          this.$message.error("上传失败");
                        }
                      })
                    }
                  }else{
                    this.$message.error(res.message);
                  }
                })
              }catch (e) {
                this.$message.error("浏览器不支持，请切换IE尝试");
                console.log(e.toString());
                return;
              }
              this.loadData(1)
            }else{
              var contaInfoList=info.file.response.result;
              this.$message.info("下列箱号无对应箱型，请填写")
              this.$refs.ManifestNanjingAddContaType.show(contaInfoList)
            }
          }
          else{
            this.$message.error(`${info.file.response.message}`);
          }
        }
        else if (info.file.status === 'error') {
          this.$message.error("上传失败")
        }
      },

      handleDelete(record){
        var deleteId=record.id;
        var oldSignXml="";
        var newSignXml="";
        getAction(`/manifestNanjing/getDeleteXml`,{
          id: record.id,
          guid: record.guid,
          customsCode: record.customsCode,
          shipId: record.shipId,
          shipName: record.shipName,
          voyageNo: record.voyageNo,
          billNo: record.billNo,
          lineFlag: record.lineFlag
        }).then(res => {
          try{
            var oldXml=res.result.oldXml;
            var newXml=res.result.newXml;
            var orgCode=Vue.ls.get('Login_Userinfo').orgCode;
            console.log(orgCode);
            getAction(`/manifestNanjing/getWhetherSign`,{orgCode:orgCode}).then(res=>{
              if(res.success){
                if(res.result==='0'){
                  oldSignXml=this.getSignedXml(oldXml);
                  if(oldSignXml===undefined || oldSignXml === '' || oldSignXml===null){
                    this.$message.error("检查是否插入USBKEY；或是否安装行助手")
                    return;
                  }
                  newSignXml=this.getSignedXml(newXml);
                  postAction(`/manifestNanjing/sendDeleteXML`,{
                    id: deleteId,
                    oldDeleteXml: oldSignXml,
                    newDeleteXml: newSignXml
                  }).then(res=>{
                    if(res.success){
                      this.$message.success("已发送删除请求");
                    }else{
                      this.$message.success("删除请求发送失败");
                    }
                  })
                }else{
                  postAction(`/manifestNanjing/sendDeleteXML`,{
                    id: deleteId,
                    oldDeleteXml: oldXml,
                    newDeleteXml: newXml
                  }).then(res=>{
                    if(res.success){
                      this.$message.success("已发送删除请求");
                    }else{
                      this.$message.success("删除请求发送失败");
                    }
                  })
                }
              }
            })

          }
          catch (e) {
            this.$message.error("浏览器不支持，请切换IE尝试");
            console.log(e.toString());
            return;
          }
        })
      },


      getSignedXml(xmlData){
        if(this.flag===false){
          var cc=this.obj.getCertificateCollectionX();
          cc.CryptoInterface=3;
          cc.Load();
          this.certificate=cc.SelectCertificateDialog();
          this.flag=true;
        }
        return this.certificate.XMLSignEnveloping(xmlData);
      },


      handleNjInfo() {
        this.$refs.ManifestNanjingInfoStatusModal.getCaptchaImg()
        this.$refs.ManifestNanjingInfoStatusModal.show()
      },

      addManifest(){
        this.$refs.ManifestNanjingAddModal.add ();
      },

      editManifest(record){
        this.$refs.ManifestNanjingAddModal.edit (record);
      },

      downLoad(record) {
        console.log(record)
        postAction(`/download/deleteXML`, {
          id: record.id,
          guid: record.guid,
          shipId: record.shipId,
          shipName: record.shipName,
          voyageNo: record.voyageNo,
          billNo: record.billNo,
          packNo: record.packNo,
          grossWeight: record.grossWeight,
          lineFlag: record.lineFlag,
          manifestSend: record.manifestSend
        }).then(res => {
          this.downloadTemplate(res.message)
        })
      },
      downloadTemplate: function(res) {
        let url = `${window._CONFIG['domianURL']}/${this.url.templateUrl}?path=${res}`
        window.location.href = url
      },


    }
  }
</script>


<style scoped>
  @import '~@assets/less/common.less'
</style>