<template>
  <a-layout>
    <a-layout>
      <a-layout-sider>
        <a-list item-layout="horizontal" :data-source="userList">
          <template #renderItem="{ item }">
            <a-list-item>
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
            <div v-for="(item, index) in records" :key="index">
              <div
                v-if="
                  (item.chatType === 'PUBLIC' && isGroup) ||
                  (
                    !isGroup &&
                    item.chatType !== 'PUBLIC' &&
                    item.senderName === currentChatUserName
                  )
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
            </div>
          </div>
        </div>
        <div class="input-box">
          <div class="chat-edit">
            <div class="chat-edit-input">
              <div>
                <textarea
                  v-model="content"
                  :rows="6"
                  placeholder="请输入内容"
                  @keyup.enter="sendMessage"
                />
              </div>
            </div>
            <div class="chat-edit-footer">按Enter发送，Ctrl+Enter换行</div>
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
  name: 'Room',
  data() {
    return {
      userList: [],
      userNumber: 0,
      isGroup: true,
      currentChatUserName: '',
      public: PUBLIC,
      personal: PERSONAL,
      content: '',
      records: [
        {
          senderName: 'test1',
          chatType: 'PUBLIC',
          message: 'test msg',
        },
        {
          senderName: 'test1',
          chatType: 'PUBLIC',
          message: 'test msg',
        },
        {
          senderName: 'test1',
          chatType: 'PERSONAL',
          message: 'test msg',
        },
      ],
    }
  },
  methods: {
    updateCurrentUser(userList, userNumber) {
      this.userList = userList
      this.userNumber = userNumber
    },
    updateRecords(newMessageObj) {
      this.records.push(newMessageObj)
    },
    sendMessage() {
      console.log('msg')
      if (this.content === '' || this.content === null) {
        message.warning('Please input message!')
        return
      }
      const chatType = this.isGroup ? PUBLIC: PERSONAL
      this.$emit(
        'send-event',
        chatType,
        this.content,
        this.currentChatUserName
      )
      this.content = ''
    }
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
.chat-edit-items {
  height: 40px;
  padding: 0 20px;
}
.chat-edit-input {
  height: 120px;
  overflow: hidden;
}
.chat-edit-footer {
  height: 30px;
  padding: 0 20px;
  text-align: right;
  color: #bcbcbc;
}
</style>