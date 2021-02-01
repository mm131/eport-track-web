<template>
  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭">

    <a-card :bordered="false">
      <detail-list title="南京港">
        <detail-list-item term="箱号">{{taskInfo.boxNo}}</detail-list-item>
        <detail-list-item term="箱型">{{taskInfo.boxType}}</detail-list-item>
        <detail-list-item term="出港号">{{taskInfo.outShipName}}</detail-list-item>
        <detail-list-item term="尺寸">{{taskInfo.boxSize}}</detail-list-item>
        <detail-list-item term="箱ISO">{{taskInfo.boxIso}}</detail-list-item>
        <detail-list-item term="箱重">{{taskInfo.boxWeight}}</detail-list-item>
        <detail-list-item term="提单号">{{taskInfo.billNo}}</detail-list-item>
        <detail-list-item term="箱主">{{taskInfo.boxOwner}}</detail-list-item>
        <detail-list-item term="高度">{{taskInfo.boxHeight}}</detail-list-item>
        <detail-list-item term="装货港">{{taskInfo.facilityId}}</detail-list-item>
        <detail-list-item term="第一卸货港">{{taskInfo.podCode}}</detail-list-item>
        <detail-list-item term="目的港">{{taskInfo.finalPodCode}}</detail-list-item>
        <detail-list-item term="航次">{{taskInfo.exVoyage}}</detail-list-item>
        <detail-list-item term="箱位分类">{{taskInfo.positionCategory}}</detail-list-item>
        <detail-list-item term="进场时间">{{taskInfo.inYardTime}}</detail-list-item>
        <detail-list-item term="海关放行时间">{{taskInfo.njCustomReleaseTime}}</detail-list-item>
        <detail-list-item term="出场时间">{{taskInfo.outYardTime}}</detail-list-item>
      </detail-list>
      <a-divider style="margin-bottom: 32px"/>
      <detail-list title="上海港 【提单信息】">
        <a-row>
          <detail-list-item term="提单">{{taskInfo.billNo}}</detail-list-item>
          <detail-list-item term="船公司">{{taskInfo.shipCompanyName}}</detail-list-item>
          <detail-list-item term="预录入时间">{{taskInfo.boxPrerecordTime}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="进场时间">{{taskInfo.arrivalPortTime}}</detail-list-item>
          <detail-list-item term="运抵报文时间">{{taskInfo.arrivalMessageSendTime}}</detail-list-item>
          <detail-list-item term="海关放行时间">{{taskInfo.shCustomReleaseTime}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="码头放行时间">{{taskInfo.portReleaseTime}}</detail-list-item>
          <detail-list-item term="出场时间">{{taskInfo.leavePortTime}}</detail-list-item>
        </a-row>
      </detail-list>
      <a-divider style="margin-bottom: 32px"/>
      <detail-list title="上海港 【海关总署舱单】">
        <a-row>
          <detail-list-item term="提运单号">{{taskInfo.billNo}}</detail-list-item>
          <detail-list-item term="关区代码">{{taskInfo.customCode}}</detail-list-item>
          <detail-list-item term="运输方式">{{taskInfo.transportation}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="提运单类型">{{taskInfo.billType}}</detail-list-item>
          <detail-list-item term="运输工具名称">{{taskInfo.transportationName}}</detail-list-item>
          <detail-list-item term="航班/航次">{{taskInfo.voyage}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="进出境日期">{{taskInfo.entryDepatureTime}}</detail-list-item>
          <detail-list-item term="件数">{{taskInfo.itemCount}}</detail-list-item>
          <detail-list-item term="重量">{{taskInfo.weight}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="进口抵港确报标志">{{taskInfo.importArrivalSignal}}</detail-list-item>
          <detail-list-item term="理货状态">{{taskInfo.tallyStatus}}</detail-list-item>
          <detail-list-item term="出口运抵状态">{{taskInfo.exportArrivalStatus}}</detail-list-item>
        </a-row>
      </detail-list>
      <a-divider style="margin-bottom: 32px"/>
      <detail-list title="上海港 【船舶信息】">
        <a-row>
          <detail-list-item term="船舶">{{taskInfo.englishVessel}}</detail-list-item>
          <detail-list-item term="出口航次">{{taskInfo.leaveVoyno}}</detail-list-item>
          <detail-list-item term="停靠码头">{{taskInfo.dockCode}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="预计抵锚地时间">{{taskInfo.anticipatedAnchorageSiteTime}}</detail-list-item>
          <detail-list-item term="计划靠泊时间">{{taskInfo.planArrivalPortTime}}</detail-list-item>
          <detail-list-item term="计划离泊时间">{{taskInfo.planLeavePortTime}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="开港时间">{{taskInfo.boxLeaveStartTime}}</detail-list-item>
          <detail-list-item term="截港时间">{{taskInfo.boxLeaveEndTime}}</detail-list-item>
          <detail-list-item term="实际抵锚地时间">{{taskInfo.actualAnchorageTime}}</detail-list-item>
        </a-row>
        <a-row>
          <detail-list-item term="实际靠泊时间">{{taskInfo.factArrivalPortTime}}</detail-list-item>
          <detail-list-item term="实际离泊时间">{{taskInfo.factLeavePortTime}}</detail-list-item>
        </a-row>
      </detail-list>
    </a-card>
  </a-modal>
</template>

<script>
  import PageLayout from '@/components/page/PageLayout'
  import DetailList from '@/components/tools/DetailList'
  import { getAction } from '@/api/manage'
  const DetailListItem = DetailList.Item

  export default {
    name: 'TaskInfoModal',
    components: {
      PageLayout,
      DetailList,
      DetailListItem
    },
    data() {
      return {
        visible:false,
        taskInfo: {},
        title () {
          return '任务详情'
        }
      }
    },
    created(){
    },
    methods: {
      loadTaskInfo(taskId) {
        this.taskInfo.id = taskId;
        return new Promise(resolve => {
          let url = "/track/task/info"
          getAction(url, {id: taskId}).then(res=>{
            if(res.success){
              this.taskInfo = res.result;
            }
            resolve()
          })
        })
      },
      returnPrevPage() {
        this.$router.back();
      },
      show() {
        this.taskInfo = {};
        this.visible = true;
      },
      handleOk(){
        this.visible = false;
      }
    },
    /*computed: {
      title () {
        return '任务详情'
      }
    }*/
  }
</script>

<style lang="scss" scoped>
  .title {
    color: rgba(0,0,0,.85);
    font-size: 16px;
    font-weight: 500;
    margin-bottom: 16px;
  }
</style>