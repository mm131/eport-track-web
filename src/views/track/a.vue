<template>

</template>

<script>
  export default {
    name: 'a.vue',
    methods: {
      pay: function(e) {
        wx.login({
          success: function(info) {
            console.log('code=' + info.code)
            wx.request({
              method: 'post',
              url: 'http://172.20.10.3:9091/wxPay/login/' + info.code,
              success: function(res) {
                wx.request({
                  url: 'http://172.20.10.3:9091/wxPay/pay/' + res.data.openid,
                  method: 'post',
                  success: function(res) {
                    wx.requestPayment({
                      timeStamp: res.data.timeStamp,
                      nonceStr: res.data.nonceStr,
                      package: res.data.package,
                      signType: res.data.signType,
                      paySign: res.data.paySign,
                      success: function(payRes) {
                        console.log('成功', payRes.errMsg)
                      },
                      fail: function(payRes) {
                        console.log('失败', payRes.errMsg)
                      }
                    })
                  }
                })
              }
            })
          }
        })
      }
    }
  }
</script>

<style scoped>

</style>