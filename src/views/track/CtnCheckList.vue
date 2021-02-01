<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

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
              <a-button type="primary" @click="searchCtnCheck" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style="border-top: 5px">
      <p>{{this.getLabelDisplay}}</p>
    </div>

    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        :scroll="{ x: 'max-content' }"
        @change="handleTableChange">
      </a-table>
    </div>
    <!-- table区域-end -->
  </a-card>
</template>

<script>
  import Vue from 'vue'
  import Storage from 'vue-ls'
  import { ACCESS_TOKEN } from '@/store/mutation-types'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JInput from '@/components/jeecg/JInput'
  import { getAction, postAction } from '@/api/manage'

  export default {
    name: 'CtnCheckList',
    mixins: [JeecgListMixin],
    components: {
      JInput
    },
    data() {
      return {
        tokenHeader: { 'X-Access-Token': Vue.ls.get(ACCESS_TOKEN) },
        description: '海事查验',
        queryParam: {},
        captchaImg: null,
        columns: [
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
            title: '查验时间',
            align: 'center',
            dataIndex: 'operateTime'
          },
          {
            title: '船名',
            align: 'center',
            dataIndex: 'vesselName'
          },
          {
            title: '被抽查单位',
            align: 'center',
            dataIndex: 'companyName'
          }
        ],
        url: {
          list: '/ctn/check/list'
        }
      }
    },
    mounted() {
      this.getCaptchaImg()
    },
    computed: {
      getLabelDisplay: function() {
        return this.ipagination.total > 0 ? `当前有 ${this.ipagination.total} 个货箱正在查验` : '';
      }
    },
    methods: {
      searchCtnCheck() {
        this.getCaptchaImg();
        if (!this.queryParam.code) {
          this.$message.warning(`验证码为必填`);
          return;
        }
        this.searchQuery();
        this.queryParam.code = '';
      },
      getCaptchaImg() {
        getAction(`/track/group/captcha`, {}).then(res => {
          if (res.success) {
            this.captchaImg = res.result.img
            this.queryParam.jid = res.result.jid
            this.queryParam.cid = res.result.cid
          }
        })
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>