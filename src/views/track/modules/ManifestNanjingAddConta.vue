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
      <a-card :bordered="true">
        <a-row :gutter="36">
          <a-col :span="24">
            <div class="table-page-search-wrapper">
              <a-form layout="inline">
                <a-row :gutter="36">

                  <a-col :md="12" :sm="12">
                    <a-form-item  label="集装箱号">
                      <a-input placeholder="请输入集装箱号" v-model="contaInfo.contaId" />
                    </a-form-item>
                  </a-col>

                  <a-col :md="12" :sm="12">
                    <a-form-item  label="集装箱型">
                      <a-input placeholder="请输入集装箱型" v-model="contaInfo.characteristiccode" />
                    </a-form-item>
                  </a-col>

                  <a-col :md="12" :sm="24">
                    <a-form-item  label="备注">
                      <a-input placeholder="请输入备注" v-model="contaInfo.notes" />
                    </a-form-item>
                  </a-col>
                  <a-col :md="8" :sm="12">
                    <a-button  @click="addConta" style="margin-left: 20px">添加</a-button>
                  </a-col>
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
                 <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record)">
                    <a>删除</a>
                 </a-popconfirm>
              </span>
            </a-table>
          </a-col>
        </a-row>
      </a-card>
    </div>
  </a-modal>
</template>



<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { postAction,getAction } from '@/api/manage'
  import ACol from 'ant-design-vue/es/grid/Col'
  export default {
    name: "ManifestNanjingAddConta",
    mixins: [JeecgListMixin],
    components: {
      ACol
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
            title: '集装箱号',
            dataIndex: 'contaId',
          },
          {
            title: '集装箱型',
            dataIndex: 'characteristiccode',
          },
          {
            title: '备注',
            dataIndex: 'notes',
          },
          {
            title: '操作',
            dataIndex: 'action',
            width: '150px',
            scopedSlots: { customRender: 'action' }
          }
        ],
        title:"操作",
        contaInfo:{},
        queryParam:{},
        dataSource:[],
        visible:false,
        confirmLoading: false,
      }
    },

    methods: {
      add (record) {
        this.contaInfo.manifestNanjingSendId=record.id;
        this.contaInfo.guid=record.guid;
        this.contaInfo.shipId=record.shipId;
        this.contaInfo.shipName=record.shipName;
        this.contaInfo.voyageNo=record.voyageNo;
        this.contaInfo.billNo=record.billNo;
        getAction(`/manifestNanjing/getContaByManifestId`,{manifestNanjingSendId:record.id}).then(res=>{
          this.dataSource=res.result;
        });
        this.visible = true;
      },

      addConta(){
        postAction(`/manifestNanjing/addConta`,this.contaInfo).then(res=>{
          getAction(`/manifestNanjing/getContaByManifestId`,{manifestNanjingSendId:this.contaInfo.manifestNanjingSendId}).then(res=>{
            this.dataSource=res.result;
          });
          this.contaInfo.contaId=null;
          this.contaInfo.characteristiccode=null;
          this.contaInfo.notes=null;
        })
      },

      handleDelete(record){
        getAction(`/manifestNanjing/deleteConta`,{id:record.id}).then(res=>{
          if(res.success){
            getAction(`/manifestNanjing/getContaByManifestId`,{manifestNanjingSendId:this.contaInfo.manifestNanjingSendId}).then(res=>{
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
