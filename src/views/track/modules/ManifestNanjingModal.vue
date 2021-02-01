<template>

  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    centered
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">

    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchUnitLogs">
        <a-row :gutter="24">
          <div>
            <a-textarea :rows="18" v-model="manifestSend"  readonly/>
          </div>
        </a-row>
      </a-form>
    </div>
  </a-modal>

</template>

<script>
  export default {
    name: 'ManifestNanjingModal',
    data() {
      return {
        visible:false,
        title:"",
        id:"",
        manifestSend:""
      }
    },
    created(){
    },
    methods: {
      loadShipInfo(entity) {
        this.id=entity.id;
        this.manifestSend=entity.manifestSend;
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
        this.close()
      }
    }
  }
</script>


<style scoped="scoped">
</style>

