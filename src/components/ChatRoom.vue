<template>
  Room
  <div>Online People Number: {{userNumber}}</div>
  <div v-for="userName in userList" v-bind:key="userName">
    {{userName}}
  </div>
</template>

<script>
export default {
  name: 'Room',
  data() {
    return {
      userList: [],
      userNumber: 0,
    }
  },
  methods: {
    updateCurrentUser(userList, userNumber) {
      this.userList = userList
      this.userNumber = userNumber
    }
  }
}
</script>
