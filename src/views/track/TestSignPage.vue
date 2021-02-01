<template>
  <div class="table-page-search-wrapper">
    <a-form layout="inline">
      <a-row :gutter="24">
        <a-col :md="16" :sm="16" style="padding-right: 0">
          <div style="border:1px solid #ccc;padding-left: 55px">
            <a-textarea :rows="18" v-model="testString" style="padding-left: 10px;border: none;border-left: 1px solid #ddd;margin:0;border-radius:0;white-space:nowrap"/>
          </div>
        </a-col>
      </a-row>
    </a-form>
    <a-button type="primary" @click="handleAddSign">加签</a-button>
    <a-divider type="vertical"/>
    <a-button type="primary" @click="handleRemoveSign">解签</a-button>
  </div>
</template>

<script>
  import Vue from 'vue'

  export default {
    name: 'TestSignPage',

    data() {
      return {
        title:"测试加签",
        testString:''
      }
    },
    mounted(){

      let obj = document.getElementsByTagName("object");
      let flag = false;
      for (let i = 0; i < obj.length; i++) {
        if (obj[i].getAttribute("id") == "WebSign") return (flag = true);
      }
      if (!flag) {
        let body = document.getElementsByTagName("body")[0];
        this.domAppend(body,'WebSign');
      }

      this.testString='<?xml version="1.0" encoding="UTF-8"?>\n' +
        '<MESSAGE>\n' +
        '<MESSAGE_HEAD>\n' +
        '<GUID>2308134766802_20200909104326192</GUID>\n' +
        '<MESSAGE_TYPE>IKA</MESSAGE_TYPE>\n' +
        '<I_E_FLAG>E</I_E_FLAG>\n' +
        '<MESSAGE_CREATE_TIME>2020-09-09 10:43:26</MESSAGE_CREATE_TIME>\n' +
        '<SENDER>2308134766802</SENDER>\n' +
        '<RECEIVER>2308</RECEIVER>\n' +
        '</MESSAGE_HEAD>\n' +
        '<MESSAGE_LIST>\n' +
        '<MANIFEST_HEAD>\n' +
        '<GUID>2308134766802_20200909104326192</GUID>\n' +
        '<SHIP_ID>FC0006375</SHIP_ID>\n' +
        '<SHIP_NAME_EN>KAITONG11</SHIP_NAME_EN>\n' +
        '<VOYAGE_NO>20632S</VOYAGE_NO>\n' +
        '<BILL_NO>ONEYNK9GZ0567800</BILL_NO>\n' +
        '<PACK_NO>249</PACK_NO>\n' +
        '<GROSS_WT>2900.14</GROSS_WT>\n' +
        '<TRN_MODE>3</TRN_MODE>\n' +
        '<LINE_FLAG>2</LINE_FLAG>\n' +
        '</MANIFEST_HEAD>\n' +
        '<MANIFEST_HEAD>\n' +
        '<GUID>2308134766802_20200909104326192</GUID>\n' +
        '<SHIP_ID>FC0006375</SHIP_ID>\n' +
        '<SHIP_NAME_EN>KAITONG11</SHIP_NAME_EN>\n' +
        '<VOYAGE_NO>20632S</VOYAGE_NO>\n' +
        '<BILL_NO>ONEYNK9GZ0632800</BILL_NO>\n' +
        '<PACK_NO>178</PACK_NO>\n' +
        '<GROSS_WT>1034.9</GROSS_WT>\n' +
        '<TRN_MODE>3</TRN_MODE>\n' +
        '<LINE_FLAG>2</LINE_FLAG>\n' +
        '</MANIFEST_HEAD>\n' +
        '<MANIFEST_HEAD>\n' +
        '<GUID>2308134766802_20200909104326192</GUID>\n' +
        '<SHIP_ID>FC0006375</SHIP_ID>\n' +
        '<SHIP_NAME_EN>KAITONG11</SHIP_NAME_EN>\n' +
        '<VOYAGE_NO>20632S</VOYAGE_NO>\n' +
        '<BILL_NO>ONEYNK9GZ0633900</BILL_NO>\n' +
        '<PACK_NO>123</PACK_NO>\n' +
        '<GROSS_WT>1790.88</GROSS_WT>\n' +
        '<TRN_MODE>3</TRN_MODE>\n' +
        '<LINE_FLAG>2</LINE_FLAG>\n' +
        '</MANIFEST_HEAD>\n' +
        '<MANIFEST_HEAD>\n' +
        '<GUID>2308134766802_20200909104326192</GUID>\n' +
        '<SHIP_ID>FC0006375</SHIP_ID>\n' +
        '<SHIP_NAME_EN>KAITONG11</SHIP_NAME_EN>\n' +
        '<VOYAGE_NO>20632S</VOYAGE_NO>\n' +
        '<BILL_NO>ONEYNK9GZ0634300</BILL_NO>\n' +
        '<PACK_NO>100</PACK_NO>\n' +
        '<GROSS_WT>1456</GROSS_WT>\n' +
        '<TRN_MODE>3</TRN_MODE>\n' +
        '<LINE_FLAG>2</LINE_FLAG>\n' +
        '</MANIFEST_HEAD>\n' +
        '<MANIFEST_CONTA>\n' +
        '<CONTA_ID>KKTU7990660</CONTA_ID>\n' +
        '<PART_BILL_NO>ONEYNK9GZ0567800</PART_BILL_NO>\n' +
        '</MANIFEST_CONTA>\n' +
        '<MANIFEST_CONTA>\n' +
        '<CONTA_ID>KKTU7990660</CONTA_ID>\n' +
        '<PART_BILL_NO>ONEYNK9GZ0632800</PART_BILL_NO>\n' +
        '</MANIFEST_CONTA>\n' +
        '<MANIFEST_CONTA>\n' +
        '<CONTA_ID>KKTU7990660</CONTA_ID>\n' +
        '<PART_BILL_NO>ONEYNK9GZ0633900</PART_BILL_NO>\n' +
        '</MANIFEST_CONTA>\n' +
        '<MANIFEST_CONTA>\n' +
        '<CONTA_ID>KKTU7990660</CONTA_ID>\n' +
        '<PART_BILL_NO>ONEYNK9GZ0634300</PART_BILL_NO>\n' +
        '</MANIFEST_CONTA>\n' +
        '</MESSAGE_LIST>\n' +
        '</MESSAGE>\n'
    },
    methods: {
      handleAddSign(){
        var addSign= WebSign.SignXMLString(this.testString);
        console.log("加签结果："+ addSign);
        this.testString=addSign;
      },

      handleRemoveSign(){
        var removeSign=WebSign.VerifyXmlString(this.testString);
        console.log("解签结果："+ addSign);
        this.testString=removeSign;
      },


      setAttr:function (attr, val) {
        let obj = document.createAttribute(attr);
        obj.value = val;
        return obj;
      },

      domAppend(body, id) {
        let obj = document.createElement("OBJECT");
        let objId = ''
        if (id) {
          objId = this.setAttr("id", id);
        } else {
          objId = this.setAttr("id", "ZJRCEP");

        }
        obj.setAttributeNode(objId);
        let objClassid = this.setAttr(
          "classid",
          "clsid:63A9768B-9997-4D89-A6F2-BDCC579354FC"//你电脑安装的ocx控件的ID
        );

        obj.setAttributeNode(objClassid);

        let codebase = this.setAttr("codebase", "WebSign.ocx#Version=1,0,1,11") //如果电脑中没有ocx此属性会在IE浏览器下自动提示下载
        obj.setAttributeNode(codebase);

        let objWidth = this.setAttr("width", 0);
        obj.setAttributeNode(objWidth);
        let objHeight = this.setAttr("height", 0);
        obj.setAttributeNode(objHeight);
        let objhspace = this.setAttr("hspace", 0);
        obj.setAttributeNode(objhspace);
        let objvspace = this.setAttr("vspace", 0);
        obj.setAttributeNode(objvspace);
        body.appendChild(obj);
      }

    }

  }
</script>

<style scoped>

</style>