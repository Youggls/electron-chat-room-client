<template>
<login v-on:login-event="login"></login>
</template>

<script>
import dgram from 'dgram'
import Login from './components/Login.vue'
import {CRLF, LOGIN, LOGIN_SUCCESS, PROTOCOL_STRING} from './constants.js'
import { message } from "ant-design-vue"

export default {
  name: 'App',
  components: {
    Login
  },
  mounted() {
    this.listenSocket = dgram.createSocket('udp4')
    this.listenSocket.on('message', (msg, rinfo) => {
      let decodedMsg = new TextDecoder(process.env.ENCODING).decode(msg)
      decodedMsg = decodedMsg.replace(PROTOCOL_STRING, '')
      const commandAndParams = decodedMsg.split(CRLF)
      const command = commandAndParams[0]
      const params = commandAndParams.slice(1)
      if (command === LOGIN) {
        this.handleLogin(params)
      }
      console.log(rinfo)
    })    
  },
  methods: {
    login(serverAddress, serverPort, userName) {
      const msg = Buffer.from(PROTOCOL_STRING + LOGIN + CRLF + userName)
      this.listenSocket.send(msg, serverPort, serverAddress, (err) => {
        console.log(err)
      })
    },
    handleLogin(params) {
      console.log(params)
      const status = params[0]
      const errMsg = params[1]
      if (status === LOGIN_SUCCESS) {
        message.success(errMsg)
      } else {
        message.error(errMsg)
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
