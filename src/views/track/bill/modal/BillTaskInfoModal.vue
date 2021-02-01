<template>

  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭">

    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchUnitLogs">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="提单号">
              <a-input placeholder="输入提单号查询" v-model="queryParam.billNo"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="6">
            <a-form-item label="验证码">
              <a-input placeholder="输入验证码" v-model="queryParam.code"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="3" :sm="6">
            <img style="margin-left: 20px;" :src="'data:image/png;base64,'+captchaImg" @click="getCaptchaImg()"/>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchUnitLogs" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <div>
      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="billNo"
        :columns="columns"
        :dataSource="dataSource"
        :loading="loading"
        @change="handleTableChange">
      </a-table>

    </div>

  </a-modal>
</template>

<script>
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { getAction, postAction } from '@/api/manage'

  export default {
    name: 'BillTaskInfoModal',
    mixins: [JeecgListMixin],
    components: {
      JInput
    },
    data() {
      return {
        visible: false,
        taskInfo: {},
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        description: '箱查询',
        queryParam: {},
        title() {
          return '任务详情'
        },
        captchaImg: null,
        cid: null,
        jid: null,
        code: '',
        timer: null,
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
            title: '提单号',
            align: 'center',
            dataIndex: 'billNo'
          },
          {
            title: '箱号',
            align: 'center',
            dataIndex: 'boxNo'
          },
          {
            title: '出门时间',
            align: 'center',
            dataIndex: 'outTime'
          },
          {
            title: '进门时间',
            align: 'center',
            dataIndex: 'inTime'
          },
          {
            title: '装船时间',
            align: 'center',
            dataIndex: 'loadTime'
          }
        ],
        isorter: {
          column: '',
          order: ''
        },
        url: {
          list: '/unitLog/list'
        }
      }
    },
    mounted() {
      this.getCaptchaImg()
    },
    methods: {
      searchUnitLogs() {
        this.getCaptchaImg()
        if (!this.queryParam.billNo || !this.queryParam.code) {
          this.$message.warning(`提单号 和 验证码为必填`)
          return
        }
        this.searchQuery()
        this.queryParam.code = ''
      },
      getCaptchaImg() {
        getAction(`/track/group/captcha`, {}).then(res => {
          if (res.success) {
            this.captchaImg = res.result.img
            this.queryParam.jid = res.result.jid
            this.queryParam.cid = res.result.cid
          }
        })
      },
      //打开弹窗时执行
      loadBillTaskInfo(taskId) {
        this.queryParam.billNo = taskId
        // console.log(taskId)
        /*this.taskInfo.id = taskId
        return new Promise(resolve => {
          let url = '/bill/task/info'
          getAction(url, { id: taskId }).then(res => {
            if (res.success) {
              this.taskInfo = res.result
            }
            resolve()
          })
        })*/
      },
      show() {
        this.taskInfo = {}
        this.visible = true
      },
      handleOk() {
        this.visible = false
        //关闭的时候重新加载表格
        this.loadData()
      }
    }
  }
</script>

<style scoped>
  @import '~@assets/less/common.less'
</style>