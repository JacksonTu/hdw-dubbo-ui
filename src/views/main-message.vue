<template>
  <div>
    <el-popover v-if="messageCount > 0" placement="bottom" trigger="hover">
      <el-badge :value="messageCount" slot="reference">
        <i class="el-icon-bell"></i>
      </el-badge>
      <div>
        <div v-for="messageContent in messageContents"  class="popitem"
             style="max-width: 228px;width: 100%;" @click="smallReadOne(messageContent)" :key="messageContent.id">
          <div style="overflow:hidden;white-space: nowrap;text-overflow: ellipsis;line-height: 1.5em;color: #606266;width: 100%;">
            {{messageContent.content}}
          </div>
          <div style="font-size: 12px; color: #999;width: 100%;text-align: right;padding-top: 6px;">
            {{messageContent.timeStr}}
          </div>
        </div>
        <el-divider></el-divider>
        <div style="font-weight: bold;width: 100%;" v-if="messageCount > 5">
          <div>
            <div @click="smallReadFive" class="popitem"
                 style="width: 50%;float: left;text-align: left;">
              标记当前为已读
            </div>
            <div @click="bigMsg" class="popitem"
                 style="width: 50%;float: left;text-align: right;">
              查看全部
            </div>
          </div>
        </div>
      </div>
    </el-popover>
    <i v-else="" class="el-icon-bell" @click="bigMsg"></i>
    <!--消息大弹框-->
    <big-message v-if="bigMsgVisible" ref="bigMsgVisible" @bigRead="bigMsgRead"></big-message>
  </div>
</template>

<script>
  import BigMessage from './main-bigMessage'
  import {getUUID} from '@/utils'
  export default {
    data () {
      return {
        bigMsgVisible: false, // 消息大弹框
        messageHide: false,
        messageContents: [],  // 最新5条消息
        messageCount: 0, // 未读消息数目
        webSocketOne: null,
        webSocketFive: null
      }
    },
    components: {
      BigMessage
    },
    created () {
      this.initWebSocketFive()
      this.initWebSocketOne()
    },
    destroyed () {
      this.overFive()
      this.overOne()
    },
    computed: {
      userId: {
        get () { return this.$store.state.user.id }
      }
    },
    methods: {
      // 大消息框回调已读
      bigMsgRead (list) {
        console.log(list)
        for (var i in list) {
          this.webSocketSend(list[i])
        }
      },
      // 显示消息大弹框
      bigMsg () {
        this.bigMsgVisible = true
        this.$nextTick(() => {
          this.$refs.bigMsgVisible.getUnread()
        })
      },
      // 已读单条
      smallReadOne (item) {
        this.webSocketSend(item.id)
        this.smallMsg(item)
      },
      // 已读当前5条
      smallReadFive () {
        if (this.messageContents) {
          var s = ''
          for (var i in this.messageContents) {
            if (i === this.messageContents.length - 1) {
              s += this.messageContents[i].id
            } else {
              s += this.messageContents[i].id
              s += ','
            }
          }
          this.webSocketSend(s)
        }
      },
      // 显示消息小弹框
      smallMsg (item) {
        this.$notify.warning({
          title: item.title,
          dangerouslyUseHTMLString: true,
          // message: '<div>' + item.content + '</div>',
          message: item.content,
          duration: 0,
          position: 'bottom-right'
        })
      },
      initWebSocketFive () {
        console.log('5条消息推送：用户ID:' + this.userId)
        const userId = this.userId + '_' + getUUID()
        const wsUri = window.SITE_CONFIG.wsUrl + '/ws/homeSms/' + userId
        this.webSocketFive = new WebSocket(wsUri)
        this.webSocketFive.onopen = function () {
          console.log('five-message-WebSocket连接成功')
        }
        this.webSocketFive.onerror = function () {
          console.log('five-message-WebSocket连接发生错误')
        }
        this.webSocketFive.onmessage = this.webSocketOnMessageFive
        this.webSocketFive.onclose = function () {
          console.log('five-message-WebSocket连接--关闭')
        }
        this.overFive = () => {
          this.webSocketFive.close()
        }
      },
      initWebSocketOne () {
        console.log('1条消息推送：用户ID:' + this.userId)
        const userId = this.userId + '_' + getUUID()
        const wsUri = window.SITE_CONFIG.wsUrl + '/ws/sms/' + userId
        this.webSocketOne = new WebSocket(wsUri)
        this.webSocketOne.onopen = function () {
          console.log('new-one-message-WebSocket连接成功')
        }
        this.webSocketOne.onerror = function () {
          console.log('new-one-message-WebSocket连接发生错误')
        }
        this.webSocketOne.onmessage = this.webSocketOnMessageOne
        this.webSocketOne.onclose = function () {
          console.log('new-one-message-WebSocket连接--关闭')
        }
        this.overOne = () => {
          this.webSocketOne.close()
        }
      },
      webSocketOnMessageFive (e) {
        const redata = JSON.parse(e.data)
        console.log('5条消息--', redata)
        this.messageContents = []
        if (redata.list) {
          this.messageContents = redata.list
        }
        if (redata.count) {
          this.messageCount = redata.count
        } else {
          this.messageCount = 0
        }
      },
      webSocketOnMessageOne (e) {
        const redata = JSON.parse(e.data)
        console.log('1条消息--', redata)
        this.smallMsg(redata)
      },
      // WebSocketfive 发送数据
      webSocketSend (agentData) {
        this.webSocketFive.send(agentData)
      }
    }
  }
</script>

<style scoped>
  .popitem {
    padding: 8px;cursor: pointer;
  }
</style>
