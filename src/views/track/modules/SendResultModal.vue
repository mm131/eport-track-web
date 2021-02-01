<template>
  <a-modal
    :title=title
    :width=width
    :visible=visible
    @ok="handleOk"
    @cancel="handleOk"
    cancelText="关闭"
    style="margin-top: 10%; ">
    <div style="font-size: larger">{{result}}</div>
  </a-modal>
</template>

<script>
  export default {
    name: 'SendResult',
    data() {
      return {
        title: '补发',
        width: 400,
        visible: false,
        result: ''
      }
    },
    methods: {
      loadSendResult(res) {
        if (res.result.substring(10, res.result.length - 2) === '操作成功') {
          this.result = '操作成功！'
        } else {
          this.result = '操作失败：' + res.result.substring(10, res.result.length - 2)
        }
      },
      loadBatchResult(res) {
        for (var m = 0; m < res.result.length; m++) {
          this.result += '箱号：' + res.result[m].split('&')[0] + ',' + '补发结果：' + res.result[m].split('&')[1].substring(10, res.result[m].split('&')[1].length - 2) + ';'
        }
      },
      show() {
        this.visible = true
      },
      handleOk() {
        this.visible = false
        this.result = ''
      }
    }
  }
</script>

<style scoped>

</style>