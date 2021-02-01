<template>
  <a-modal
    :title="title"
    :width="1100"
    :visible="visible"
    :confirmLoading="confirmLoading"
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
              <a-form-item  label="航线标志">
                <a-select v-model="manifestInfo.lineFlag" placeholder="请选择航线">
<!--                  <a-select-option value="1">干线(1)</a-select-option>-->
                  <a-select-option value="2">支线(2)</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="运输工具编号">
                <a-input placeholder="请输入运输工具编号" v-model="manifestInfo.shipId" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="船名">
                <a-input placeholder="请输入船名"  v-model="manifestInfo.shipName"  />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item  label="航次">
                <a-input placeholder="请输入航次"  v-model="manifestInfo.voyageNo" />
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
              <a-form-item label="箱货类型">
                <a-switch checked-children="集装箱货" un-checked-children="非集装箱货" default-checked @change="onChange"/>
              </a-form-item>
            </a-col>

            <a-col :md="8" :sm="12">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button @click="addTemp">暂存</a-button>
              <a-button type="primary"  style="margin-left: 8px">保存并申报</a-button>
            </span>
            </a-col>
          </a-row>

        </a-form>
      </div>


      <a-card :bordered="true">
        <a-row :gutter="36">
          <a-col :span="24">
            <div class="table-page-search-wrapper">
              <a-form layout="inline">
                <a-row :gutter="36">

                  <a-col :md="8" :sm="12">
                    <a-form-item  label="提运单号">
                      <a-input placeholder="请输入提运单号" v-model="manifestInfo.billNo" />
                    </a-form-item>
                  </a-col>

                  <a-col :md="8" :sm="12">
                    <a-form-item  label="件数">
                      <a-input placeholder="请输入件数" v-model="manifestInfo.packNo" />
                    </a-form-item>
                  </a-col>

                  <a-col :md="8" :sm="12">
                    <a-form-item  label="毛重">
                      <a-input placeholder="请输入毛重" v-model="manifestInfo.grossWeight" />
                    </a-form-item>
                  </a-col>

                  <a-col :md="8" :sm="12">
                    <a-form-item  label="出口转关方式">
                      <a-select v-model="manifestInfo.trnMode" placeholder="请选择出口转关方式">
                        <a-select-option value="1">提前报关(2)</a-select-option>
                        <a-select-option value="2">批量中转(3)</a-select-option>
                        <a-select-option value="3">关区内中转(9)</a-select-option>
                      </a-select>
                    </a-form-item>
                  </a-col>

                  <a-button  @click="addManifest" style="margin-left: 20px">添加</a-button>
                </a-row>

              </a-form>
            </div>
            <a-table
              ref="table"
              bordered
              rowKey="id"
              size="middle"
              :columns="columns"
              :pagination= false
              :dataSource="dataSource"
              @change="handleTableChange">
              <span slot="action" slot-scope="text, record">
                 <a href="javascript:;" @click="addConta(record)" v-if="isShow">填加集装箱</a>
                 <a-divider type="vertical" v-if="isShow"/>
                 <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record)">
                    <a>删除</a>
                 </a-popconfirm>
              </span>
            </a-table>
          </a-col>
        </a-row>
      </a-card>
    </div>
    <manifest-nanjing-add-conta ref="ManifestNanjingAddConta"></manifest-nanjing-add-conta>
  </a-modal>
</template>



<script>
  import Vue from 'vue'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { postAction,getAction } from '@/api/manage'
  import ManifestNanjingAddConta from './ManifestNanjingAddConta'
  export default {
    name: "ManifestNanjingAddModal",
    mixins: [JeecgListMixin],
    components: {
      ManifestNanjingAddConta
    },
    data() {
      return {
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
            title: '提运单号',
            dataIndex: 'billNo',
          },
          {
            title: '件数',
            dataIndex: 'packNo',
          },
          {
            title: '毛重',
            dataIndex: 'grossWeight',
          },
          {
            title: '出口转关方式',
            dataIndex: 'trnMode',
          },
          {
            title: '操作',
            dataIndex: 'action',
            width: '150px',
            scopedSlots: { customRender: 'action' }
          }
        ],
        title:"操作",
        manifestInfo:{orgCode:Vue.ls.get('Login_Userinfo').orgCode,lineFlag:"2",trnMode:"2"},
        queryParam:{},
        dataSource:[],
        visible:false,
        confirmLoading: false,
        isShow:true
      }
    },

    mounted(){
      this.manifestInfo.guid=null;
    },

    methods: {
      add () {
        // this.manifestInfo={};
        this.visible = true;
      },
      edit (record) {
        this.manifestInfo.lineFlag=record.lineFlag;
        this.manifestInfo.shipId=record.shipId;
        this.manifestInfo.shipName=record.shipName;
        this.manifestInfo.voyageNo=record.voyageNo;
        this.manifestInfo.guid=record.guid;
        getAction(`/manifestNanjing/getManifetByGuid`,{guid:record.guid}).then(res=>{
          this.dataSource=res.result;
        });
        this.visible = true;

      },

      addTemp(){
        this.visible=false;
      },

      onChange(checked) {
        if(checked===true){
          this.isShow=true;
        }else{
          this.isShow=false;
        }
      },


      addManifest(){
        postAction(`/manifestNanjing/manifestInsert`,this.manifestInfo).then(res=>{
            this.manifestInfo.guid=res.result;
            getAction(`/manifestNanjing/getManifetByGuid`,{guid:res.result}).then(res=>{
              this.dataSource=res.result;
            });
          this.manifestInfo.billNo=null;
          this.manifestInfo.packNo=null;
          this.manifestInfo.grossWeight=null;
          this.manifestInfo.trnMode=null;
        })
      },

      addConta(record){
        this.$refs.ManifestNanjingAddConta.add (record);
      },

      handleDelete(record){
        getAction(`/manifestNanjing/deleteManifest`,{id:record.id}).then(res=>{
          if(res.success){
            getAction(`/manifestNanjing/getManifetByGuid`,{guid:record.guid}).then(res=>{
              this.dataSource=res.result;
            });
          }
        })
      },


      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        this.$emit('showdata',this.listData);
        this.close();
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
</style>
