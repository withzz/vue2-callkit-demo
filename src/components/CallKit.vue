<template>
  <div>
    <p style="color: red;">{{ message }}</p>
    呼叫组件简易demo，使用步骤如下：
    1、输入appkey等账号信息
    2、后点击登录
    3、呼叫对方或者收到对方呼叫之后点击接听
    <div>
      <button @click="login">登录</button>
      <button @click="call">呼叫</button>
      <button @click="accept">接听</button>
      <button @click="hangup">挂断</button>
    </div>
    <div>
      appkey:
      <input type="text" v-model="appkey" />
      accId:
      <input type="text" v-model="accId" />
      token:
      <input type="text" v-model="token" />
      callee:
      <input type="text" v-model="callee" />
    </div>
    <div id="NE_local" style="display: inline-block;float: left;width:320px; height:240px">本地视图</div>
    <div id="NE_remote" style="display: inline-block;float: right;width:320px; height:240px">远端视图</div>
  </div>
</template>

<script>
import NIM from "@yxim/nim-web-sdk/dist/SDK/NIM_Web_NIM"
import { NECall } from '@xkit-yx/call-kit'

const neCall = NECall.getInstance()

export default {
  name: 'CallKit',
  data() {
    return {
      appkey: '',
      accId: '',
      token: '',
      callee: '',
      message: '',
    }
  },
  methods: {
    login() {
      const im = NIM.getInstance({
        appKey: this.appkey, // im appkey
        token: this.token, // im token
        account: this.accId, // im account
        logLevel: 'debug',
        onconnect: () => {
          neCall.setup({
            nim: im, // im 实例用户需要先初始化
            appkey: this.appkey, // 应用的 appKey，G2 平台的 appKey
            debug: true, // 是否需要开启日志，默认开启
            rtcConfig: {
              videoResolution: 'VIDEO_QUALITY.VIDEO_QUALITY_720p',
              videoFrameRate: 'VIDEO_FRAME_RATE.CHAT_VIDEO_FRAME_RATE_NORMAL',
              audioQuality: 'speech_low_quality',
            }, // rtcConfig 用来设置 RTC 可选。相关参数参照 可以参考[G2官方文档](https://doc.yunxin.163.com/docs/jcyOTA0ODM/zUxMDQzOTM?platformId=50082)
          })
          // 来电监听
          neCall.on('onReceiveInvited', (value) => {
            // 用户可以触发视图，比如弹起邀请页面
            console.log(value)
            this.message = `收到 ${value.callerAccId} 呼叫，可以点击接听按钮了`

          })
          // 通话结束的回调。通话被取消、拒绝、通话中挂断，即所有导致通话结束的行为都会触发此回调
          neCall.on('onCallEnd', (value) => {
            console.log(value)
            this.message = ''
          })
          // 话单回调
          neCall.on('onRecordSend', (value) => {
            console.log(value)
          })
          // 主被叫加入通道的回调，通话建立后触发
          neCall.on('onCallConnected', (value) => {
            console.log(value)
          })
        },
        ondisconnect: () => {
          neCall?.destroy()
        },
      })
    },
    call() {
      // 发起呼叫
      const localView = document.getElementById('NE_local')
      const remoteView = document.getElementById('NE_remote')
      console.log(this.neCall)
      neCall.setLocalView(localView)
      neCall.setRemoteView(remoteView, this.callee)
      neCall.setTimeout(30)//设置呼叫超时取消时间，单位：秒
      neCall.call({
        accId: this.callee, // 被叫 im 的 id
        callType: '2', //呼叫类型，1表示语音通话，2表示视频通话 
      })
    },
    accept() {
      // 接受邀请
      const localView = document.getElementById('NE_local')
      const remoteView = document.getElementById('NE_remote')
      neCall.setLocalView(localView)
      neCall.setRemoteView(remoteView, this.callee)
      // 接受邀请，需要先设置视图
      neCall.accept()
      this.message = ''
    },
    hangup() {
      // 挂断
      neCall.hangup()
      this.message = ''
    }
  },
}
</script>
