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
      <div class="table-page-search-wrapper">
        <a-form layout="inline">

          <a-row :gutter="36">
            <a-col :md="8" :sm="12">
              <a-form-item  label="预录入号">
                <a-input  v-model="form.preEntryNumber" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
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
              <a-form-item  label="启运港">
                <a-input   v-model="form.startPort" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="改配原因">
                <a-input   v-model="form.reason" />
              </a-form-item>
            </a-col>
          </a-row>

          <a-row :gutter="36">
            <a-col :md="8" :sm="12">
              <a-form-item  label="内支线驳船">
                <a-input  v-model="form.branchLineName" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="驳船航次">
                <a-input   v-model="form.branchLineVoyage" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="驳船靠泊时间">
                <a-input   v-model="form.branchLineTime" />
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

          </a-row>

          <a-row :gutter="36">
            <a-col :md="16" :sm="24">
              <a-form-item  label="备注">
                <a-textarea v-model="form.remark" auto-size />
              </a-form-item>
            </a-col>
          </a-row>


        </a-form>
      </div>

    </div>

  </a-modal>
</template>



<script>
  export default {
    name: "ManifestShanghaiDistributionBeforeModal",
    components: {
    },
    data() {
      return {
        title:"操作",
        id:null,
        form:{
          preEntryNumber:"",
          billNo:"",
          counts:"",
          startPort:"",

          branchLineName:"",
          branchLineVoyage:"",
          branchLineTime:"",

          lineNameBefore:"",
          lineVoyageBefore:"",
          linePortBefore:"",
          shipPeriodBefore:"",

          lineNameNow:"",
          lineVoyageNow:"",
          linePortNow:"",
          shipPeriodNow:"",

          reason:"",
          remark:"",

          customPort:"",
          year:"",
          month:"",
          day:""

        },
        visible:false,
      }
    },


    methods: {
      loadBillInfo (record) {
        this.form={
          preEntryNumber:"",
          billNo:"",
          counts:"",
          startPort:"",

          branchLineName:"",
          branchLineVoyage:"",
          branchLineTime:"",

          lineNameBefore:"",
          lineVoyageBefore:"",
          linePortBefore:"",
          shipPeriodBefore:"",

          lineNameNow:"",
          lineVoyageNow:"",
          linePortNow:"",
          shipPeriodNow:"",

          reason:"",
          remark:"",

          customPort:"",
          year:"",
          month:"",
          day:""
        };

        var billNoString='';
        for(var i=0;i<record.length;i++){
          if(billNoString.length>0){
            billNoString=billNoString+' \r\n'+record[i].billNo;
          }else{
            billNoString=billNoString+record[i].billNo;
          }
        }
        this.form.preEntryNumber=record[0].preEntryNumber;
        this.form.billNo=billNoString;
        this.form.counts=record.length;
        this.form.startPort=record[0].startPort;
        this.form.branchLineName=record[0].branchLineName;
        this.form.branchLineVoyage=record[0].branchLineVoyage;
        this.form.lineNameBefore=record[0].lineName;
        this.form.lineVoyageBefore=record[0].lineVoyage;
        this.form.linePortBefore=record[0].linePort;

        this.form.customPort=this.form.linePortBefore.startsWith("WGQ") ? "外港" : "洋山";

        let date = new Date();
        let month=date.getMonth()+1;
        let day=date.getDate();
        this.form.year=date.getFullYear();
        this.form.month= month < 10 ? "0" + month : month;
        this.form.day=day < 10 ? "0" + day : day;

        this.visible = true;
      },




      close () {
        this.$emit('close');
        this.visible = false;
      },

      handleOk () {
        let _this = this;
        // 读取并获得模板文件的二进制内容
        var url= url="/通关前改配.docx";
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
          doc.setData({
            ..._this.form
          });

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
          saveAs(out, "通关前改配.docx");
        });

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
