<template>
    <a-modal
      :title="title"
      :width="700"
      :visible="visible"
      @ok="handleOk"
      @cancel="handleOk"
      cancelText="关闭">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="16" :sm="16">
            <a-form-item label="请选择要下载的格式">
              <a-select v-model="type" placeholder="" style="width: 200px;">
                <a-select-option value=1>YML_VGM</a-select-option>
                <a-select-option value=2>ONE_VGM</a-select-option>
                <a-select-option value=3>亿通_VGM</a-select-option>
                <a-select-option value=4>森罗_VGM</a-select-option>
                <a-select-option value=5>长荣_VGM</a-select-option>
                <a-select-option value=6>东方海外_VGM</a-select-option>
                <a-select-option value=7>汉堡南美_VGM</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col>
              <a-button type="primary" @click="download()"  icon="download">下载</a-button>
          </a-col>
        </a-row>
      </a-form>

    </a-modal>
</template>

<script>
  import Vue from 'vue'
  import JInput from '@/components/jeecg/JInput'
  export default {
    name: 'DownloadModal',
    components: {
      JInput
    },
    data() {
      return {
        visible:false,
        id:'',
        type:'',
        title:'下载',
        url: {
          downloadUrl: 'vgm/download'
        }
      }
    },

    methods: {
      showPage(id){
        this.id=id;
        this.visible=true;
      },

      download(){
        if(this.type==null || this.type==''){
          this.$message.error("请选择下载格式！");
          return;
        }
        let url=`${window._CONFIG['domianURL']}/${this.url.downloadUrl}?id=${this.id}&type=${this.type}`;
        console.log(url);
        window.location.href=url;
      },

      handleOk(){
        this.visible=false;
      }
    }
  }
</script>

<style scoped>

</style>