<template>
  <div class="container">
    <div class="userinfo">
      <img :src="userinfo.avatarUrl"
           alt="">
      <p>{{userinfo.nickName}}</p>
    </div>
    <YearProgress></YearProgress>
    <button v-if='userinfo.openId'
            @click="scanBook"
            class="btn">添加图书</button>
    <button v-else
            open-type="getUserInfo"
            lang="zh_CN"
            class='btn'
            @getuserinfo="login">点击登录</button>
  </div>
</template>

<script>
import qcloud from 'wafer2-client-sdk'
import config from '@/config'
import { showSuccess, showModal, post } from '@/util'

import YearProgress from '@/components/YearProgress'

export default {
  data() {
    return {
      userinfo: {
        avatarUrl: 'http://image.shengxinjing.cn/rate/unlogin.png',
        nickName: ''
      }
    }
  },
  onShow() {
    let userinfo = wx.getStorageSync('userinfo')
    if (userinfo) {
      this.userinfo = userinfo
    }
    console.log(this.userinfo)
  },
  methods: {
    async addBook(isbn) {
      const res = await post('/weapp/addbook', {
        isbn,
        openId: this.userinfo.openId
      })
      showModal('添加成功', `${res.title}添加成功`)
    },
    scanBook() {
      wx.scanCode({
        success: res => {
          if (res.result) {
            this.addBook(res.result)
          }
          console.log(res)
        }
      })
    },
    login() {
      const session = qcloud.Session.get()
      qcloud.setLoginUrl(config.loginUrl)
      if (session) {
        // 第二次登录
        // 或者本地已经有登录态
        // 可使用本函数更新登录态
        qcloud.loginWithCode({
          success: res => {
            this.userinfo = res
            showSuccess('登录成功')
          },
          fail: err => {
            console.error(err)
            showModal('登录错误', err.message)
          }
        })
      } else {
        // 首次登录
        qcloud.login({
          success: res => {
            this.userinfo = res
            wx.setStorageSync('userinfo', res)
            showSuccess('登录成功')
          },
          fail: err => {
            console.error(err)
            showModal('登录错误', err.message)
          }
        })
      }
    }
  },
  components: {
    YearProgress
  }
}
</script>

<style lang="scss" scoped>
.container {
  padding: 0 30rpx;
}
.userinfo {
  margin-top: 100rpx;
  text-align: center;
  img {
    width: 150rpx;
    height: 150rpx;
    margin: 20rpx;
    border-radius: 50%;
  }
}
</style>
