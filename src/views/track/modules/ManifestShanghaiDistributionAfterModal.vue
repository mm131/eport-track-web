<template>
  <a-modal
    :title="title"
    :width="1000"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
    wrapClassName="ant-modal-cust-warp"
  >
    <div class="home">

      <div v-if="type===2" style="padding-bottom: 20px;">
        <table border="1" class="JzxTable">
          <tr>
            <th style="width: 40%;">预录入号</th>
            <th style="width: 40%;">报关单号</th>
            <th style="width: 40%;">提单号</th>
          </tr>
          <tr v-for="(contaInfo,index) in tableData">
            <td>{{contaInfo.preEntryNumber}}</td>
            <td>{{contaInfo.customsDeclarationNumber}}</td>
            <td>{{contaInfo.billNo}}</td>
          </tr>
        </table>
      </div>

      <div class="table-page-search-wrapper">
        <a-form layout="inline">

          <a-row :gutter="36">
            <a-col :md="8" :sm="12" v-if="type===1 ||type==3">
              <a-form-item  label="预录入号">
                <a-input  v-model="form.preEntryNumber" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12" v-if="type===1 ||type==3">
              <a-form-item  label="报关单号">
                <a-input   v-model="form.customsDeclarationNumber"  />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12" v-if="type===1 ||type==3">
              <a-form-item  label="提单号">
                <a-input   v-model="form.billNo" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="改配提单票数">
                <a-input   v-model="form.counts" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="20型号箱量">
                <a-input   v-model="form.counts1" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="40型号箱量">
                <a-input   v-model="form.counts2" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="12">
              <a-form-item  label="箱主">
                <a-input   v-model="form.boxOwner" />
              </a-form-item>
            </a-col>
          </a-row>

          <a-row :gutter="36">
            <a-col :md="8" :sm="12">
              <a-form-item  label="原船名">
                <a-input   v-model="form.lineNameBefore" />
              </a-form-item>
            </a-col>


            <a-col :md="8" :sm="12">
              <a-form-item  label="航次">
                <a-input   v-model="form.lineVoyageBefore" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="码头">
                <a-input   v-model="form.linePortBefore" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="船期">
                <a-input   v-model="form.shipPeriodBefore" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="12">
              <a-form-item  label="船IMO号">
                <a-input   v-model="form.imoBefore" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="干线船代">
                <a-input   v-model="form.companyBefore" />
              </a-form-item>
            </a-col>
          </a-row>

          <a-row :gutter="36">
            <a-col :md="8" :sm="12">
              <a-form-item  label="现改配船名">
                <a-input   v-model="form.lineNameNow" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="航次">
                <a-input   v-model="form.lineVoyageNow" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="码头">
                <a-input   v-model="form.linePortNow" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="船期">
                <a-input   v-model="form.shipPeriodNow" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="船IMO号">
                <a-input   v-model="form.imoNow" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="干线船代">
                <a-input   v-model="form.companyNow" />
              </a-form-item>
            </a-col>
          </a-row>

          <a-row :gutter="36">
            <a-col :md="8" :sm="12">
              <a-form-item  label="改配原因">
                <a-input   v-model="form.reason" />
              </a-form-item>
            </a-col>
          </a-row>

        </a-form>
      </div>

    </div>

  </a-modal>
</template>



<script>
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import { getAction } from '@/api/manage'
  export default {
    name: "ManifestShanghaiDistributionAfterModal",
    mixins: [JeecgListMixin],
    components: {
    },
    data() {
      return {
        title:"操作",
        type:null,
        id:null,
        form:{
          preEntryNumber:"",
          customsDeclarationNumber:"",
          billNo:"",
          counts:"",
          counts1:"",
          counts2:"",
          lineNameBefore:"",
          lineVoyageBefore:"",
          linePortBefore:"",
          shipPeriodBefore:"",
          imoBefore:"",
          lineNameNow:"",
          lineVoyageNow:"",
          linePortNow:"",
          shipPeriodNow:"",
          imoNow:"",
          companyBefore:"",
          boxOwner:"",
          reason:"",
          companyNow:""
        },
        visible:false,
        tableData: [],
        chosenData: [],
        billNoArr:"",
        lineVoyageArr:"",
        downloadUrl: 'distribution/download'
      }
    },


    methods: {
      loadContaInfo (record,type) {
        this.type=type;
        this.tableData=[];
        this.form={
            preEntryNumber:"",
            customsDeclarationNumber:"",
            billNo:"",
            counts:"",
            counts1:"",
            counts2:"",
            lineNameBefore:"",
            lineVoyageBefore:"",
            linePortBefore:"",
            shipPeriodBefore:"",
            imoBefore:"",
            lineNameNow:"",
            lineVoyageNow:"",
            linePortNow:"",
            shipPeriodNow:"",
            imoNow:"",
            companyBefore:"",
            boxOwner:"",
            reason:"",
            companyNow:""
        };
        if(type===1){
          this.getChosenFaList(record.billNo,record.lineVoyage)
          this.id=record.id;
          this.form.preEntryNumber=record.preEntryNumber;
          this.form.customsDeclarationNumber=record.customsDeclarationNumber;
          this.form.billNo=record.billNo;
          this.form.lineNameBefore=record.lineName;
          this.form.lineVoyageBefore=record.lineVoyage;
          this.form.linePortBefore=record.linePort;
          this.form.boxOwner=record.boxOwner;
          this.billNoArr=record.billNo;
          this.lineVoyageArr=record.lineVoyage;
          this.form.counts=1;

        }
        if(type===2) {
          this.tableData=record;
          var billNoString='';
          var lineVoyageString='';
          for(var i=0;i<record.length;i++){
            if(billNoString.length>0){
              billNoString=billNoString+','+record[i].billNo;
              lineVoyageString=lineVoyageString+','+record[i].lineVoyage;
            }else{
              billNoString=billNoString+record[i].billNo;
              lineVoyageString=lineVoyageString+record[i].lineVoyage;
            }
          }
          this.billNoArr=billNoString;
          this.lineVoyageArr=lineVoyageString;
          this.getChosenFaList(billNoString,lineVoyageString)
          this.form.preEntryNumber=record[0].preEntryNumber;
          this.form.customsDeclarationNumber=record[0].customsDeclarationNumber;
          this.form.billNo=record[0].billNo;
          this.form.lineNameBefore=record[0].lineName;
          this.form.lineVoyageBefore=record[0].lineVoyage;
          this.form.linePortBefore=record[0].linePort;
          this.form.boxOwner=record[0].boxOwner;
          this.form.counts=record.length;
        }
        if(type===3){
          var billNoString='';
          var lineVoyageString='';
          for(var i=0;i<record.length;i++){
            if(billNoString.length>0){
              billNoString=billNoString+','+record[i].billNo;
              lineVoyageString=lineVoyageString+','+record[i].lineVoyage;
            }else{
              billNoString=billNoString+record[i].billNo;
              lineVoyageString=lineVoyageString+record[i].lineVoyage;
            }
          }
          this.billNoArr=billNoString;
          this.lineVoyageArr=lineVoyageString;
          this.getChosenFaList(billNoString,lineVoyageString)
          this.id=record[0].id;
          this.form.preEntryNumber=record[0].preEntryNumber;
          this.form.customsDeclarationNumber=record[0].customsDeclarationNumber;
          this.form.billNo=billNoString;
          this.form.lineNameBefore=record[0].lineName;
          this.form.lineVoyageBefore=record[0].lineVoyage;
          this.form.linePortBefore=record[0].linePort;
          this.form.boxOwner=record[0].boxOwner;
          this.form.counts=record.length;
        }

        this.visible = true;
      },


      getChosenFaList(billNo,lineVoyage){
        getAction(`/distribution/getChosenFaList`,{
          billNo: billNo,
          lineVoyage: lineVoyage
        }).then(res => {
          if(res.success){
            this.chosenData=res.result;
            this.form.counts1=this.chosenData.filter(item => item.boxSize === "20").length;
            this.form.counts2=this.chosenData.filter(item => item.boxSize === "40").length;
          }else{
            this.$message.error("获取数据失败");
          }
        })
      },




      close () {
        this.$emit('close');
        this.visible = false;
      },

      handleOk () {
        let _this = this;
        // 读取并获得模板文件的二进制内容
        var url="";
        if(_this.type==1 || _this.type==3){
          url="/通关后改配_单票.docx";
        }else{
          url="/通关后改配_多票.docx";
        }
        JSZipUtils.getBinaryContent(url, function(error, content) {
          // input.docx是模板。我们在导出的时候，会根据此模板来导出对应的数据
          // 抛出异常
          if (error) {
            throw error;
          }
          // 创建一个JSZip实例，内容为模板的内容
          let zip = new JSZip(content);
          let doc = new window.docxtemplater().loadZip(zip);
          // 设置模板变量的值
          if(_this.type==1 || _this.type==3){
            doc.setData({
              ..._this.form
            });
          }else{
            doc.setData({
              ..._this.form,
              table: _this.tableData
            });
          }
          try {
            // 用模板变量的值替换所有模板变量
            doc.render();
          } catch (error) {
            // 抛出异常
            let e = {
              message: error.message,
              name: error.name,
              stack: error.stack,
              properties: error.properties
            };
            console.log(JSON.stringify({ error: e }));
            throw error;
          }
          // 生成一个代表docxtemplater对象的zip文件（不是一个真实的文件，而是在内存中的表示）
          let out = doc.getZip().generate({
            type: "blob",
            mimeType:
              "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
          });
          saveAs(out, "通关后改配.docx");
        });

        this.batchDownload();
      },

      batchDownload(){
        let url=`${window._CONFIG['domianURL']}/${this.downloadUrl}?billNoArr=${this.billNoArr}&lineVoyageArr=${this.lineVoyageArr}&companyBefore=${this.form.companyBefore}&imoBefore=${this.form.imoBefore}&transitPortNow=${this.form.transitPortNow}&lineNameNow=${this.form.lineNameNow}&lineVoyageNow=${this.form.lineVoyageNow}&linePortNow=${this.form.linePortNow}&imoNow=${this.form.imoNow}&companyNow=${this.form.companyNow}&shipPeriodBefore=${this.form.shipPeriodBefore}&shipPeriodNow=${this.form.shipPeriodNow}`;
        window.location.href=url;
      },



      handleCancel () {
        this.close()
      },
    }

  }
</script>

<style lang="scss">
  .table-page-search-wrapper {
    .ant-form-inline {
      .ant-form-item {
        display: flex;
        margin-bottom: 24px;
        margin-right: 0;

        .ant-form-item-control-wrapper {
          flex: 1 1;
          display: inline-block;
          vertical-align: middle;
        }

        > .ant-form-item-label {
          line-height: 32px;
          padding-right: 8px;
          width: auto;
        }
        .ant-form-item-control {
          height: 32px;
          line-height: 32px;
        }
      }
    }

    .table-page-search-submitButtons {
      display: block;
      margin-bottom: 24px;
      white-space: nowrap;
    }
  }

  .JzxTable{
    width: 100%;
    text-align: center;
    margin-top: 5px;
  }
  .JzxTable th{
    padding: 7px 0;
  }
  .JzxTable td{
    padding: 4px 0;
  }
</style>
