<template>
  <login v-if="showLogin" v-on:login-event="login"></login>
  <room ref="chatRoom" v-if="showRoom" v-on:send-event="sendMessage"></room>
</template>

<script>
import dgram from 'dgram'
import Login from './components/Login.vue'
import Room from './components/ChatRoom.vue'
import {
  CHECK,
  CHECK_TIME,
  CRLF,
  FALSE,
  LOGIN,
  MESSAGE,
  PROTOCOL_STRING,
  PUBLIC,
  REPLY_MSG,
  TRUE,
} from './constants.js'
import { message } from 'ant-design-vue'
import { ref } from 'vue'

export default {
  name: 'App',
  components: {
    Login,
    Room,
  },
  data() {
    return {
      showLogin: true,
      showRoom: false,
      serverAddress: '',
      serverPort: 0,
      userName: '',
    }
  },
  setup() {
    const chatRoom = ref()
    const updateCurrentUser = (userList, userNumber, userName) => {
      chatRoom.value.updateCurrentUser(userList, userNumber, userName)
    }
    const updateRecords = (newMessageObject) => {
      chatRoom.value.updateRecords(newMessageObject)
    }
    return { chatRoom, updateCurrentUser, updateRecords }
  },
  mounted() {
    this.listenSocket = dgram.createSocket('udp4')
    this.listenSocket.on('message', (msg, rinfo) => {
      const [command, params] = this.parseRawData(msg)
      if (command === LOGIN) {
        this.handleLogin(params)
      } else if (command === CHECK) {
        this.handleCheck(params)
      } else if (command === MESSAGE) {
        this.handleMessage(params)
      } else if (command === REPLY_MSG) {
        this.handleReplyMessage(params)
      }
      console.log(rinfo)
    })
    setInterval(() => {
      this.sendCheck()
    }, CHECK_TIME)
  },
  methods: {
    login(serverAddress, serverPort, userName) {
      const msg = Buffer.from(PROTOCOL_STRING + LOGIN + CRLF + userName)
      this.serverAddress = serverAddress
      this.serverPort = serverPort
      this.userName = userName
      this.listenSocket.send(msg, serverPort, serverAddress, (err) => {
        if (err !== null) {
          message.error(err)
        }
      })
    },
    // Parse raw data
    parseRawData(data) {
      let decodedMsg = new TextDecoder(process.env.ENCODING).decode(data)
      const [command, ...params] = decodedMsg
        .replace(PROTOCOL_STRING, '')
        .split(CRLF)
      return [command, params]
    },
    // handler function
    handleLogin(params) {
      const [status, errMsg] = params
      if (status === TRUE) {
        message.success(errMsg)
        // Switch to chat room
        this.showLogin = false
        this.showRoom = true
        this.sendCheck()
      } else {
        message.error(errMsg)
      }
    },
    handleCheck(params) {
      const status = params[0]
      if (status === TRUE) {
        const [userNumber, userListStr] = params.slice(1)
        const userList = userListStr.split('\n')
        const userName = this.userName
        this.updateCurrentUser(userList, userNumber, userName)
      } else {
        message.error('You are offline!')
      }
    },
    handleMessage(params) {
      const [senderName, chatType, message] = params
      const messageObject = {
        senderName: senderName,
        chatType: chatType,
        message: message,
      }
      this.updateRecords(messageObject)
    },
    handleReplyMessage(params) {
      const [targetUser, status, errMsg] = params
      if (status === FALSE) {
        message.error(targetUser + errMsg)
      }
    },
    // sender function
    sendCheck() {
      const msg = Buffer.from(PROTOCOL_STRING + CHECK + CRLF + this.userName)
      this.listenSocket.send(
        msg,
        this.serverPort,
        this.serverAddress,
        (err) => {
          if (err) message.error(err)
        }
      )
    },
    sendMessage(chatType, messageDetail, receiverName = null) {
      let param_receive = PUBLIC
      if (chatType !== PUBLIC) {
        param_receive = receiverName
      }
      const msg = Buffer.from(
        PROTOCOL_STRING +
          MESSAGE +
          CRLF +
          this.userName +
          CRLF +
          param_receive +
          CRLF +
          messageDetail
      )
      this.listenSocket.send(
        msg,
        this.serverPort,
        this.serverAddress,
        (err) => {
          if (err) message.error(err)
        }
      )
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
