<template>
  <a-card :bordered="false">
    <a-modal
      :title="title"
      :width="550"
      :visible="visible"
      @ok="handleOk"
      @cancel="handleNotOk"
      cancelText="关闭">

      <span>以下集装箱缺少箱型,请补填尺寸和型号,示例：20GP</span>
      <table border="1" class="JzxTable">
        <tr>
          <th style="width: 10%;">序号</th>
          <th style="width: 50%;">集装箱号</th>
          <th style="width: 40%;">集装箱箱型</th>
        </tr>
        <tr v-for="(contaInfo,index) in contaInfoList">
          <td>{{index+1}}</td>
          <td>{{contaInfo.contaId}}</td>
          <td><input name="contaType"></td>
        </tr>
      </table>
    </a-modal>
  </a-card>
</template>

<script>
  import {getAction,postAction } from '@/api/manage'
  import Vue from 'vue'


  export default {
    name: 'ManifestNanjingAddContaType',
    components: {
    },
    data() {
      return {
        title() {
          return '集装箱箱型补填'
        },
        visible: false,
        confirmLoading: false,
        contaInfoList: [],
        obj:[],
        flag: false,
        certificate:[],
      }
    },

    mounted(){
      this.obj=new NetONEX();
      this.obj.setupObject();
    },

    methods: {
      show(contaInfoList) {
        this.contaInfoList=contaInfoList;
        var contaTypeList=document.getElementsByName('contaType');
        for(var i=0;i<contaTypeList.length;i++){
         contaTypeList[i].value="";
        }
        this.visible = true;
      },
      handleNotOk() {
        this.visible = false
      },
      handleOk() {
        var contaTypeList= document.getElementsByName('contaType');
        for(var i=0;i<contaTypeList.length;i++){
          this.contaInfoList[i].contaType=contaTypeList[i].value;
        }
        postAction(`/manifestNanjing/addContaType`,{contaTypes: JSON.stringify(this.contaInfoList)}).then(res => {
          if(res.success){
            var orgCode=Vue.ls.get('Login_Userinfo').orgCode;
            var oldXML=res.result.oldXml;
            var newXML=res.result.newXml;
            var oldSignXML="";
            var newSignXml="";
            try{
              getAction(`/manifestNanjing/getWhetherSign`,{orgCode:orgCode}).then(res=>{
                if(res.success){
                  if(res.result==='0'){      //需要加签
                    oldSignXML=this.getSignedXml(oldXML);
                    if(oldSignXML===undefined || oldSignXML === '' || oldSignXML===null){
                      this.$message.error("检查是否插入USBKEY；或是否安装行助手")
                      return;
                    }
                    newSignXml=this.getSignedXml(newXML);
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
            this.visible = false

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

    }
  }
</script>


<style scoped>
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