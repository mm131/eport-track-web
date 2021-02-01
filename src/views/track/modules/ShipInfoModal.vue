<template>
  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭">

    <a-card :bordered="false">
      <detail-list title="船舶信息">
        <detail-list-item term="英文船名">{{shipInfo.shipNameEn}}</detail-list-item>
        <detail-list-item term="中文船名">{{shipInfo.localName}}</detail-list-item>
        <detail-list-item term="船舶mmsi">{{shipInfo.mmsi}}</detail-list-item>
        <detail-list-item term="船舶imo">{{shipInfo.imono}}</detail-list-item>
        <detail-list-item term="船舶编号">{{shipInfo.shipNO}}</detail-list-item>
        <detail-list-item term="IC卡号">{{shipInfo.icno}}</detail-list-item>
        <detail-list-item term="船舶呼号">{{shipInfo.callsign}}</detail-list-item>
        <detail-list-item term="船舶类型">{{shipInfo.shipTypeCode}}</detail-list-item>
        <detail-list-item term="国籍">{{shipInfo.flagCode}}</detail-list-item>
        <detail-list-item term="船籍港">{{shipInfo.registrationPort}}</detail-list-item>
        <detail-list-item term="船舶所有人">{{shipInfo.owner}}</detail-list-item>
        <detail-list-item term="联系人电话">{{shipInfo.contactNO}}</detail-list-item>
        <detail-list-item term="最大航速">{{shipInfo.maxSpeed}}</detail-list-item>
        <detail-list-item term="船舶长度">{{shipInfo.loa}}</detail-list-item>
        <detail-list-item term="垂直间长">{{shipInfo.lbp}}</detail-list-item>
        <detail-list-item term="船舶型深">{{shipInfo.depth}}</detail-list-item>
        <detail-list-item term="船舶型宽">{{shipInfo.bm}}</detail-list-item>
        <detail-list-item term="船舶高度">{{shipInfo.height}}</detail-list-item>
        <detail-list-item term="夏季满载吃水">{{shipInfo.draught}}</detail-list-item>
        <detail-list-item term="总吨">{{shipInfo.gross}}</detail-list-item>
        <detail-list-item term="核定抗风等级">{{shipInfo.windLoading}}</detail-list-item>
        <detail-list-item term="货舱数及总容积">{{shipInfo.hatch}}</detail-list-item>
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
    name: 'ShipInfoModal',
    components: {
      PageLayout,
      DetailList,
      DetailListItem
    },
    data() {
      return {
        visible:false,
        shipInfo: {}
      }
    },
    created(){
    },
    methods: {
      loadShipInfo(entity) {
        console.log(entity);
        this.shipInfo = entity;
      },
      returnPrevPage() {
        this.$router.back();
      },
      show() {
        // this.shipInfo = {};
        this.visible = true;
      },
      handleOk(){
        this.visible = false;
      }
    },
    computed: {
      title () {
        return '船舶详情'
      }
    }
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