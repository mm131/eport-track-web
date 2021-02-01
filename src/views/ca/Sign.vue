<template>
  <div>
    <!--<OBJECT id='WebSign' classid='CLSID:5C457383-C43E-4F0F-BACD-8CAD3CE597C5'  width='0'height='0'>
      <param name='AutoSign' value='0'>
      <param name='CertificateFilter' value='0'>
      <param name='Quiet' value='1'>
      <param name='Debug' value='0'>
    </OBJECT>
    <OBJECT id='CACertificate' classid='CLSID:5CFC9864-E612-41D0-B564-F7872562F5D7'   width='0'height='0'></OBJECT>-->
<!--
    <object classid="clsid:63A9768B-9997-4D89-A6F2-BDCC579354FC" id="WebSign" codebase="WebSign.ocx#Version=1,0,1,11" Height = 0 Width = 0>

    </object> -->
    <a-button icon="check" @click="configCheck" title="检查是否可以正确使用USBKEY" style="margin-right: 20px;">环境检查</a-button>

    <a-upload :file-list="fileList" :remove="handleRemove"  :multiple="true" :before-upload="beforeUpload"
              :showUploadList="false"
              accept=".xml"
              class="upload-list-inline">
      <a-button icon="select" :disabled="loading">选择文件</a-button>
    </a-upload>
    <a-button style="margin:0 0 0 20px" icon="upload"  @click="upload()"  :loading="loading">上传</a-button>
    <a-button style="margin:0 0 0 20px" v-show="!isStop" icon="stop" @click="stop()" title="正在或以上传的不会停止">停止</a-button>
    <div style="clear: both"></div>
    <div style="margin-top: 20px;">
      <div style="float:left;width: 49%;">

        <div style=""><h3>待处理文件列表</h3></div>
        <div style="border: 1px solid #cccccc;padding:10px;">
          <div class="ant-upload-list ant-upload-list-text"  v-for="(item,index) in fileList">
            <div class="ant-upload-list-item ant-upload-list-item-undefined">
              <div class="ant-upload-list-item-info">
            <span>
              <i aria-label="图标: paper-clip" class="anticon anticon-paper-clip">
                <svg viewBox="64 64 896 896" data-icon="paper-clip" width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
                  <path d="M779.3 196.6c-94.2-94.2-247.6-94.2-341.7 0l-261 260.8c-1.7 1.7-2.6 4-2.6 6.4s.9 4.7 2.6 6.4l36.9 36.9a9 9 0 0 0 12.7 0l261-260.8c32.4-32.4 75.5-50.2 121.3-50.2s88.9 17.8 121.2 50.2c32.4 32.4 50.2 75.5 50.2 121.2 0 45.8-17.8 88.8-50.2 121.2l-266 265.9-43.1 43.1c-40.3 40.3-105.8 40.3-146.1 0-19.5-19.5-30.2-45.4-30.2-73s10.7-53.5 30.2-73l263.9-263.8c6.7-6.6 15.5-10.3 24.9-10.3h.1c9.4 0 18.1 3.7 24.7 10.3 6.7 6.7 10.3 15.5 10.3 24.9 0 9.3-3.7 18.1-10.3 24.7L372.4 653c-1.7 1.7-2.6 4-2.6 6.4s.9 4.7 2.6 6.4l36.9 36.9a9 9 0 0 0 12.7 0l215.6-215.6c19.9-19.9 30.8-46.3 30.8-74.4s-11-54.6-30.8-74.4c-41.1-41.1-107.9-41-149 0L463 364 224.8 602.1A172.22 172.22 0 0 0 174 724.8c0 46.3 18.1 89.8 50.8 122.5 33.9 33.8 78.3 50.7 122.7 50.7 44.4 0 88.8-16.9 122.6-50.7l309.2-309C824.8 492.7 850 432 850 367.5c.1-64.6-25.1-125.3-70.7-170.9z">

                  </path>
                </svg>
              </i>
              <span :title="item.name" class="ant-upload-list-item-name">{{item.name}}</span>
              <i aria-label="图标: close" tabindex="-1" class="anticon anticon-close close-width" @click="delFile(index)"><svg viewBox="64 64 896 896" data-icon="close" width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class=""><path d="M563.8 512l262.5-312.9c4.4-5.2.7-13.1-6.1-13.1h-79.8c-4.7 0-9.2 2.1-12.3 5.7L511.6 449.8 295.1 191.7c-3-3.6-7.5-5.7-12.3-5.7H203c-6.8 0-10.5 7.9-6.1 13.1L459.4 512 196.9 824.9A7.95 7.95 0 0 0 203 838h79.8c4.7 0 9.2-2.1 12.3-5.7l216.5-258.1 216.5 258.1c3 3.6 7.5 5.7 12.3 5.7h79.8c6.8 0 10.5-7.9 6.1-13.1L563.8 512z"></path></svg></i>
            </span>
              </div>
            </div>
          </div>
        </div>

      </div>
      <div  v-show="isUpload" style="float:right;width: 49%;">
        <div><h3>处理信息</h3></div>
        <div style="border: 1px solid #cccccc;padding:10px;">
          <div>
            <span>上传进度{{fileFinish}}/{{fileTotal}}</span>
          </div>

          <div style="margin-top: 10px;">
            <div v-for="(item,index) in backMsg">
              <div v-if="item.code==1" style="color: green">{{item.name}}&nbsp;成功&nbsp;{{item.msg}}</div>
              <div v-if="item.code==2" style="color: red;">{{item.name}}&nbsp;失败&nbsp;{{item.msg}}</div>
              <div v-if="item.code==3" style="color:grey;">{{item.name}}&nbsp;处理中&nbsp;{{item.msg}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>

   <!-- <div>
      <div v-for="(item,index) in fileList">
        <div>
          <div>
            <sapn>
              <i aria-label="图标: paper-clip" class="anticon anticon-paper-clip"><svg viewBox="64 64 896 896" data-icon="paper-clip" width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class=""><path d="M779.3 196.6c-94.2-94.2-247.6-94.2-341.7 0l-261 260.8c-1.7 1.7-2.6 4-2.6 6.4s.9 4.7 2.6 6.4l36.9 36.9a9 9 0 0 0 12.7 0l261-260.8c32.4-32.4 75.5-50.2 121.3-50.2s88.9 17.8 121.2 50.2c32.4 32.4 50.2 75.5 50.2 121.2 0 45.8-17.8 88.8-50.2 121.2l-266 265.9-43.1 43.1c-40.3 40.3-105.8 40.3-146.1 0-19.5-19.5-30.2-45.4-30.2-73s10.7-53.5 30.2-73l263.9-263.8c6.7-6.6 15.5-10.3 24.9-10.3h.1c9.4 0 18.1 3.7 24.7 10.3 6.7 6.7 10.3 15.5 10.3 24.9 0 9.3-3.7 18.1-10.3 24.7L372.4 653c-1.7 1.7-2.6 4-2.6 6.4s.9 4.7 2.6 6.4l36.9 36.9a9 9 0 0 0 12.7 0l215.6-215.6c19.9-19.9 30.8-46.3 30.8-74.4s-11-54.6-30.8-74.4c-41.1-41.1-107.9-41-149 0L463 364 224.8 602.1A172.22 172.22 0 0 0 174 724.8c0 46.3 18.1 89.8 50.8 122.5 33.9 33.8 78.3 50.7 122.7 50.7 44.4 0 88.8-16.9 122.6-50.7l309.2-309C824.8 492.7 850 432 850 367.5c.1-64.6-25.1-125.3-70.7-170.9z"></path></svg></i>
              <span :title="item.name" class="ant-upload-list-item-name">{{item.name}}</span>
            </sapn>
          </div>
          <i aria-label="图标: delete" tabindex="-1" class="anticon anticon-delete" title="删除文件"><svg viewBox="64 64 896 896" data-icon="delete" width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class=""><path d="M360 184h-8c4.4 0 8-3.6 8-8v8h304v-8c0 4.4 3.6 8 8 8h-8v72h72v-80c0-35.3-28.7-64-64-64H352c-35.3 0-64 28.7-64 64v80h72v-72zm504 72H160c-17.7 0-32 14.3-32 32v32c0 4.4 3.6 8 8 8h60.4l24.7 523c1.6 34.1 29.8 61 63.9 61h454c34.2 0 62.3-26.8 63.9-61l24.7-523H888c4.4 0 8-3.6 8-8v-32c0-17.7-14.3-32-32-32zM731.3 840H292.7l-24.2-512h487l-24.2 512z"></path></svg></i>
        </div>
      </div>
    </div>-->
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    name: 'Sign',
    mixins: [],
    components: {
    },
    data() {
      return {
        description: 'CA加签',
        fileList:[],
        isUpload:false,
        isStop:true,
        fileTotal:0,
        fileFinish:0,
        fileSuccess:0,
        fileError:0,
        backMsg:[],
        initHandler:{name:'',code:3,msg:'正在处理...'},
        loading:false,
        url:{
          saveUrl:'/manifestNanjing/saveSignManifest'
        }
      }
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      handleRemove(file) {
        const index = this.fileList.indexOf(file);
        const newFileList = this.fileList.slice();
        newFileList.splice(index, 1);
        this.fileList = newFileList;
      },
      beforeUpload(file) {
        this.isUpload = false;
        var isExist = false;
        this.fileList.forEach(file1 => {
          if(file1.name===file.name){
            isExist = true;
            return;
          }
        });
        if(isExist){
          this.$message.warning(file.name+"文件名重复！");
          return false;
        }
        this.fileList = [...this.fileList, file];
        return false;
      },
      //文件预览
      handlePreview:function(file){
        console.log(file.url);
      },
      stop:function(){
        this.isStop = true;
        //this.loading = false;
        this.$message.success("停止成功，等待正在处理的结束！");
      },
      stateJud:function(state){
      },
      upload: function() {
        /*if(!this.isActivex()){
          return;
        }*/
        var $this = this;
        $this.loading = true;
        $this.fileTotal = 0;
        $this.fileFinish = 0;
        $this.backMsg = [];
        $this.isUpload = true;
        $this.isStop = false;
        $this.fileTotal = $this.fileList.length;
        $this.saveCa();
        /*$this.fileList.forEach((file,index)=>{
          var reader = new FileReader()
          reader.readAsText(file, "UTF-8");//读取文件
          reader.onload = function(evt) {
            var data = evt.target.result;
            var sign = "";
            var cer = "";
            var valueCoder = "";
            /!* var sign = WebSign.PKCS1(data, 0);
             var cer = WebSign.GetCertConext();
             var valueCoder = "";*!/
            //返回加签的xml报文 为空失败
            var signXml = WebSign.SignXMLString(data);
            if (signXml === null || signXml==="" || signXml===undefined)
            {
              $this.$message.error(WebSign.GetErrorMessage);
              return false;
            }
            else
            {
              console.log(signXml);
            }
           /!* axios({
              url: " http://localhost:8888/datasync/sources/xml",
              method: 'get',
              params: {xml:sign,cer:cer,xmlg:valueCoder}
            }).then((res)=>{
              $this.$message.success("成功！");
            });*!/
          }
        });*/



      },
      saveCa:function(){
        var $this = this;
        if($this.fileList.length>0){
          var file = $this.fileList[0];
          var item = {name:file.name,code:$this.initHandler.code,msg:$this.initHandler.msg};
          $this.backMsg.push(item);
          $this.fileList.splice(0,1);
          var urlPath=`${window._CONFIG['domianURL']}`+this.url.saveUrl;
          new Promise((resolve, reject) => {
            var reader = new FileReader()
            reader.readAsText(file, "UTF-8");//读取文件
            reader.onload = function(evt) {
              var data = evt.target.result;
              var cer = '';
              var signXml = '';
              try{
                //获取证书
                //cer = WebSign.GetCertConext();
                //返回加签的xml报文 为空失败
                signXml = WebSign.SignXMLString(data);
                if(signXml===undefined || signXml === '' || signXml===null){
                  reject('检查是否插入USBKEY；或是否安装行助手');
                  return;
                }
              }catch (e) {
                reject('浏览器不支持，请切换IE尝试');
                return;
              }

              //if(!this.isStop){
                //保存到服务器
                axios({
                  url: urlPath,
                  method: 'post',
                  params: {message:signXml}
                }).then((res)=>{
                  resolve(res);
                }).catch((res)=>{
                  reject('请求后台方法错误！');
                });
              //}else{
                //reject('操作取消！');
              //}
            }
          }).then((res)=>{
            item = {name:file.name,code:1,msg:res.data.message};
          }).catch((res)=>{
            item = {name:file.name,code:2,msg:res};
          }).finally((res)=>{
            $this.backMsgUpdate(item,$this);
            $this.fileFinish++;
            if(!this.isStop){
              $this.saveCa();
            }else{
              $this.loading = false;
            }
          });
        }else{
          $this.isStop = true;
          $this.loading = false;
        }
      },
      backMsgUpdate:function(item,$this){
        $this.backMsg.forEach((file,index)=>{
          if(file.name===item.name){
            file.name = item.name;
            file.code = item.code;
            file.msg = item.msg;
            return;
          }
        });
      },
      delFile:function(index){
        this.fileList.splice(index,1);
        if(this.fileTotal>0){
          this.fileTotal--;
        }
      },
      configCheck:function(){
        try{
          //随便使用一个函数验证，可以使用Login函数
          var dn = WebSign.GetCertDN();
          if(dn===undefined || dn==='' || dn === null){
            this.$message.warning("检查是否插入USBKEY；或是否安装行助手");
          }else{
            this.$message.success("环境检查通过");
          }
        }catch (e) {
          this.$message.warning("浏览器不支持，请切换IE尝试");
        }
      },
      configCheckMsg:function(){

      },
      isActivex:function(){
        if(this.window.ActiveXObject){
        }else{
          this.$message.warning("当前浏览器不支持CA控件读取");
          return false;
        }
        return true;
      },
      sleep:function(numberMillis){
        var now = new Date();
        var exitTime = now.getTime() + numberMillis;
        while (true) {
          now = new Date();
          if (now.getTime() > exitTime)
            return;
        }
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
    },
    mounted() {
        //this.isActivex();
        //由于页面中有多处使用到同一ocx方法，避免重复创建进行对应的判断处理创建对应的object元素
        let obj = document.getElementsByTagName("object");
        let flag = false;
        for (let i = 0; i < obj.length; i++) {
          if (obj[i].getAttribute("id") == "WebSign") return (flag = true);
        }
        if (!flag) {
          let body = document.getElementsByTagName("body")[0];
          this.domAppend(body,'WebSign');
        }
      }
  }
</script>
<style scoped>
  .ant-upload-list-item-name{
    width: auto;
    padding-right: 22px;
  }
  .close-width {
    right: auto;
  }
</style>