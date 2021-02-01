<template>

  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    centered
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchUnitLogs">
        <a-row :gutter="24">
          <a-col :md="12" :sm="12" style="padding-right: 0">
<!--            <div  contenteditable="true" style="height: 360px;overflow-x: scroll;border:1px solid #ccc;border-radius: 3px;padding: 4px;" v-html="beforeModify">-->
<!--              {{beforeModify}}-->
<!--            </div>-->
            <div style="border:1px solid #ccc;padding-left: 55px">
              <a-textarea :rows="18" v-model="beforeModify" style="padding-left: 10px;border: none;border-left: 1px solid #ddd;margin:0;border-radius:0;white-space:nowrap"/>
            </div>
          </a-col>

          <a-col :md="12" :sm="12" style="padding-left: 0">
            <div style="border:1px solid #ccc;border-left:none;padding-left: 55px">
              <a-textarea :rows="18" v-model="afterModify" style="padding-left: 10px;border: none;border-left: 1px solid #ddd;margin:0;border-radius:0;white-space:nowrap" />
            </div>
<!--            <div  contenteditable="true"  @blur="blurFun($event)"  style="height: 360px;overflow-y: scroll;border:1px solid #ccc;border-radius: 3px;padding: 4px;"  v-html="afterModify">-->
<!--                {{afterModify}}-->
<!--            </div>-->
          </a-col>
        </a-row>
      </a-form>
    </div>

    <div>
      <code-diff :old-string="beforeModify" :new-string="afterModify" :context="30" :outputFormat="showType"/>
    </div>

    <template slot="footer">
      <a-button type="dashed" @click="handleCancel">取消</a-button>
      <a-button type="primary" @click="handleOk">暂存</a-button>
      <a-button type="primary" @click="sendManifest">保存并发送</a-button>
    </template>

  </a-modal>

</template>

<script>
  import { postAction } from '@/api/manage'
  import CodeDiff from 'vue-code-diff'
  export default {
    name: 'ManifestModal',
    components: {CodeDiff},
    data() {
      return {
        visible:false,
        title:"",
        id:"",
        beforeModify:"",
        afterModify:"",
        showType:"side-by-side"
      }
    },
    created(){
    },
    methods: {
      loadShipInfo(entity) {
        this.id=entity.id;
        this.beforeModify=entity.beforeModify;
        this.afterModify=entity.afterModify;
        this.visible = true;
      },
      returnPrevPage() {
        this.$router.back();
      },


      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      },
      handleOk(){
        postAction(`/manifest/updateManifest`, {
            id:this.id,
            afterModify:this.afterModify
        }).then(res => {
          if (res.success) {
              this.$emit('ok');
              this.close();
          }else{
            this.$message.error(res.message);
          }
        })
      },
      sendManifest(){
        var that = this;
        this.$confirm({
          title: '提示',
          content: '确认发送这条舱单吗？',
          onOk() {
            postAction(`/manifest/sendManifest`, {
              id:that.id,
              afterModify:that.afterModify
            }).then(res => {
              if (res.success) {
                that.$message.success("发送成功");
              }else{
                that.$message.error("发送失败");
              }
              that.$emit('ok');
              that.close();
            })
          },
          onCancel() {},
        });
      }


    }
  }
</script>


<style scoped="scoped">
</style>

