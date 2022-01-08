<template>
  <a-layout>
    <a-layout>
      <a-layout-sider>
        <a-list item-layout="horizontal" :data-source="userList">
          <a-list-item id="PUBLIC" v-on:click="switchChatRoom('PUBLIC')">
            <a-list-item-meta>
              <template #title>
                <a style="color: #ffffff">Group Chat</a>
              </template>
              <template #avatar>
                <img
                  src="../assets/avatar.png"
                  class="avatar"
                  width="30"
                  height="30"
                />
              </template>
            </a-list-item-meta>
            <a-tag color="red" v-if="unreadCount['PUBLIC'] !== 0">
              {{ unreadCount['PUBLIC'] }}
            </a-tag>
          </a-list-item>
          <template #renderItem="{ item }">
            <a-list-item
              v-if="item !== userName"
              :id="item"
              v-on:click="switchChatRoom(item)"
            >
              <a-list-item-meta>
                <template #title>
                  <a style="color: #ffffff">{{ item }}</a>
                </template>
                <template #avatar>
                  <img
                    src="../assets/avatar.png"
                    class="avatar"
                    width="30"
                    height="30"
                  />
                </template>
              </a-list-item-meta>
              <a-tag color="red" v-if="unreadCount[item] !== 0">
                {{ unreadCount[item] }}
              </a-tag>
            </a-list-item>
          </template>
        </a-list>
      </a-layout-sider>
      <a-layout-content>
        <div v-if="isGroup" class="room-len">
          Group Chat, Online User Number: {{ userNumber }}
        </div>
        <div v-else>Private Chat. User Name: {{ currentChatUserName }}</div>
        <div class="message-box" id="scroll-box">
          <div>
            <div
              v-for="(item, index) in records[currentChatUserName]"
              :key="index"
            >
              <p class="time">
                <span class="time-span">{{ item.time }}</span>
              </p>
              <div
                v-if="
                  (item.chatType === 'PUBLIC' &&
                    isGroup &&
                    item.senderName !== userName) ||
                  (!isGroup &&
                    item.chatType !== 'PUBLIC' &&
                    item.senderName === currentChatUserName)
                "
                class="message-item"
              >
                <div class="nickname">{{ item.senderName }}</div>
                <img
                  src="../assets/avatar.png"
                  class="avatar"
                  width="30"
                  height="30"
                />
                <div class="text">
                  <span v-html="item.message" />
                </div>
              </div>
              <div
                v-if="
                  (item.chatType === 'PUBLIC' &&
                    isGroup &&
                    item.senderName === userName) ||
                  (!isGroup &&
                    item.chatType !== 'PUBLIC' &&
                    item.senderName === userName)
                "
                class="self-message-item"
              >
                <div class="nickname">{{ item.senderName }}</div>
                <div class="text" style="background-color: #70c14d">
                  <span
                    style="background-color: #70c14d"
                    v-html="item.message"
                  />
                </div>
                <img
                  src="../assets/avatar.png"
                  class="avatar"
                  width="30"
                  height="30"
                />
              </div>
            </div>
          </div>
        </div>
        <div class="input-box">
          <div class="chat-edit">
            <div class="chat-edit-input">
              <div>
                <a-textarea
                  v-model:value="content"
                  :rows="6"
                  :bordered="false"
                  placeholder="Please Input Message"
                  @keyup.enter="sendMessage"
                />
              </div>
            </div>
            <div class="chat-edit-footer">Press Enter to send message</div>
          </div>
        </div>
      </a-layout-content>
    </a-layout>
  </a-layout>
</template>

<script>
import { message } from 'ant-design-vue'
import { PERSONAL, PUBLIC } from '../constants.js'
export default {
  name: 'ChatRoom',
  data() {
    return {
      userList: [],
      userNumber: 0,
      userName: '',
      isGroup: true,
      currentChatUserName: PUBLIC,
      public: PUBLIC,
      personal: PERSONAL,
      content: '',
      records: {
        PUBLIC: [],
      },
      unreadCount: {
        PUBLIC: 0,
      },
    }
  },
  methods: {
    getNowFormatDate() {
      let date = new Date()
      const seperator1 = '-'
      const seperator2 = ':'
      let month = date.getMonth() + 1
      let strDate = date.getDate()
      if (month >= 1 && month <= 9) {
        month = '0' + month
      }
      if (strDate >= 0 && strDate <= 9) {
        strDate = '0' + strDate
      }
      const currentdate =
        date.getFullYear() +
        seperator1 +
        month +
        seperator1 +
        strDate +
        ' ' +
        date.getHours() +
        seperator2 +
        date.getMinutes() +
        seperator2 +
        date.getSeconds()
      return currentdate
    },
    updateCurrentUser(userList, userNumber, userName) {
      console.log(userList)
      this.userList = userList
      for (const user of userList) {
        if (!(user in this.records)) {
          this.records[user] = []
          this.unreadCount[user] = 0
        }
      }
      this.userNumber = userNumber
      this.userName = userName
    },
    updateRecords(newMessageObj) {
      newMessageObj['time'] = this.getNowFormatDate()
      if (newMessageObj['chatType'] === PUBLIC) {
        if (!(PUBLIC in this.records)) this.records[PUBLIC] = []
        this.records[PUBLIC].push(newMessageObj)
        if (this.currentChatUserName !== PUBLIC) this.unreadCount[PUBLIC] += 1
      } else {
        const senderName = newMessageObj['senderName']
        if (!(senderName in this.records)) this.records[senderName] = []
        this.records[senderName].push(newMessageObj)
        if (this.currentChatUserName !== senderName)
          this.unreadCount[senderName] += 1
      }
    },
    sendMessage() {
      console.log(this.userName)
      if (
        this.content === '' ||
        this.content === null ||
        this.content === '\n'
      ) {
        message.warning('Please input message!')
        this.content = ''
        return
      }
      const chatType = this.isGroup ? PUBLIC : PERSONAL
      if (!this.isGroup) {
        const record = {
          senderName: this.userName,
          chatType: chatType,
          message: this.content,
          isRead: true,
          time: this.getNowFormatDate(),
        }
        this.records[this.currentChatUserName].push(record)
      }
      this.$emit('send-event', chatType, this.content, this.currentChatUserName)
      this.content = ''
    },
    switchChatRoom(targetChatName) {
      if (targetChatName !== this.currentChatUserName) {
        if (targetChatName === PUBLIC) this.isGroup = true
        else this.isGroup = false
        this.currentChatUserName = targetChatName
        for (
          let i = 0;
          i < this.records[this.currentChatUserName].length;
          i++
        ) {
          this.records[this.currentChatUserName][i]['isRead'] = true
          this.unreadCount[this.currentChatUserName] = 0
        }
        this.$forceUpdate()
      }
    },
  },
}
</script>

<style scoped>
.room-len {
  padding: 12px;
  border-bottom: 1px solid #fff;
}
.message-box {
  height: 592px;
  overflow-y: scroll;
}
.text {
  display: inline-block;
  position: relative;
  padding: 0 10px;
  max-width: 'calc(100% - 40px)';
  min-height: 30px;
  line-height: 2.5;
  font-size: 14px;
  text-align: left;
  word-break: break-all;
  background-color: #ffffff;
  border-radius: 4px;
}
.input-box {
  width: 100%;
  margin-top: 10px;
  border-top: 1px solid #dcdfe6;
}
.message-item {
  margin-left: 10px;
  text-align: left;
}
.self-message-item {
  margin-right: 10px;
  text-align: right;
}
.chat-edit-items {
  height: 40px;
  padding: 0 20px;
}
.chat-edit-input {
  height: 120px;
}
.chat-edit-footer {
  height: 30px;
  padding: 0 20px;
  text-align: right;
  color: #bcbcbc;
}
.time {
  margin: 7px 0;
  text-align: center;
}
.time-span {
  display: inline-block;
  padding: 0 18px;
  font-size: 12px;
  color: #fff;
  border-radius: 2px;
  background-color: #dcdcdc;
}
</style>