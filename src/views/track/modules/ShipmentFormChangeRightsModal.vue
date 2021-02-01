<template>

  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    centered
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">


    <div style="margin-top: 10px;">
      <a-form layout="inline" >
        <a-form-item label="选择查看人">
          <a-select
            mode="multiple"
            style="width: 300px;"
            v-model="users"
            @change="handleChange">
            <a-select-option v-for="(item,index) in userList" :value="item.id"  :key="index">
              {{ item.realname }}
            </a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </div>

  </a-modal>

</template>

<script>

  import Vue from 'vue'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { getAction,postAction} from '@/api/manage'


  export default {
    name: 'ShipmentFormChangeRightsModal',
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        userList:{},
        visible:false,
        oldViewPermissions:'',
        viewPermissions:'',
        id:'',
        title:'转发',
        users:[],
        url: {
          importUrl: 'shipmentForm/importShipmentForm'
        }

      }
    },

    computed: {
      importUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}`
      }
    },

    methods: {
      changeRigths(record) {
        this.id=record.id;
        this.oldViewPermissions=record.viewPermissions;
        this.users=[];
        if(record.viewPermissions !==null && record.viewPermissions !==''){
          this.users=record.viewPermissions.split(',');
        }
        getAction(`/sys/user/queryUsersAll`).then(res=>{
          if(res.success){
            this.userList = res.result;
          }
        })
        this.visible=true;
      },

      handleChange(value) {
        //console.log("yayyayay"+this.users);
        this.viewPermissions=`${value}`;
      },

      close () {
        this.$emit('ok2');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      },
      handleOk: function() {
        if (this.oldViewPermissions !==null && this.oldViewPermissions !== '' && this.viewPermissions !== '') {
          var arr = this.viewPermissions.split(',')
          for (var i = 0; i < arr.length; i++) {
            if(this.oldViewPermissions.indexOf(arr[i])===-1) {
              this.oldViewPermissions= this.oldViewPermissions+ ',' + arr[i];
            }
          }
          this.viewPermissions = this.oldViewPermissions
        }
        postAction(`/shipmentForm/insertViewPermissions`, {
          viewPermissions: this.viewPermissions,
          id: this.id
        }).then(res => {
          if (res.success) {
            this.$message.success('转发成功！')
          }
        })
        this.users=[];
        this.close()
      }
    }
  }
</script>


<style scoped="scoped">
</style>

