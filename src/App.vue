<template>
  <login v-if="showLogin" v-on:login-event="login"></login>
</template>

<script>
import dgram from 'dgram'
import Login from './components/Login.vue'
import {CHECK, CRLF, LOGIN, PROTOCOL_STRING, TRUE} from './constants.js'
import { message } from "ant-design-vue"

export default {
  name: 'App',
  components: {
    Login
  },
  data() {
    return {
      showLogin: true,
      serverAddress: '',
      serverPort: 0,
      userName: '',
    }    
  },
  mounted() {
    this.listenSocket = dgram.createSocket('udp4')
    this.listenSocket.on('message', (msg, rinfo) => {
      const [command, params] = this.parseRawData(msg)
      if (command === LOGIN) {
        this.handleLogin(params)
      } else if (command === CHECK) {
        this.handleCheck(params)
      }
      console.log(rinfo)
    })
  },
  methods: {
    login(serverAddress, serverPort, userName) {
      const msg = Buffer.from(PROTOCOL_STRING + LOGIN + CRLF + userName)
      this.serverAddress = serverAddress
      this.serverPort = serverPort
      this.userName = userName
      this.listenSocket.send(msg, serverPort, serverAddress, (err) => {
        console.log(err)
      })
    },
    parseRawData(data) {
      let decodedMsg = new TextDecoder(process.env.ENCODING).decode(data)
      const [command, ...params] = decodedMsg.replace(PROTOCOL_STRING, '').split(CRLF)
      return [command, params]
    },
    handleLogin(params) {
      const [status, errMsg] = params
      if (status === TRUE) {
        message.success(errMsg)
        // Switch to chat room
        this.showLogin = false
        this.sendCheck()
      } else {
        message.error(errMsg)
      }
    },
    sendCheck() {
      const msg = Buffer.from(PROTOCOL_STRING + CHECK + CRLF + this.userName)
      this.listenSocket.send(msg, this.serverPort, this.serverAddress, (err) => {
        console.log(err)
      })
    },
    handleCheck(params) {
      const status = params[0]
      if (status === TRUE) {
        const [userNumber, userListStr] = params.slice(1)
        const userList = userListStr.split('\n')
        console.log('Current online users number: ', userNumber)
        console.log('Current online users: ', userList)
      } else {
        message.error('You are offline!')
      }
    }
  }
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
