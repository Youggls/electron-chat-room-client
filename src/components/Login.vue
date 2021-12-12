<template>
  <a-form>
    <a-form-item label="Server Address">
      <a-input
        type="text"
        v-model:value="serverAddress"
        placeholder="Server Address"
      />
    </a-form-item>
    <a-form-item label="Server Port">
      <a-input type="text" v-model:value="serverPort" placeholder="8989" />
    </a-form-item>
    <a-form-item label="User Name">
      <a-input
        type="text"
        v-model:value="userName"
        placeholder="Your User Name"
      />
    </a-form-item>
  </a-form>
  <a-button type="primary" v-on:click="loginButton()"> Connect </a-button>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      serverAddress: '127.0.0.1',
      serverPort: 8989,
      userName: 'Alice',
    }
  },
  mounted() {},
  methods: {
    loginButton() {
      this.$emit(
        'login-event',
        this.serverAddress,
        this.serverPort,
        this.userName
      )
    },
  },
}
</script>

