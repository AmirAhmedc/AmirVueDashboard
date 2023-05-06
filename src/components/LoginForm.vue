<template>
  <div class="container">
    <h1 class="text-center">Login</h1>
    <div class="row justify-content-center">
      <div class="col-sm-6">
        <form @submit.prevent="login">
          <div class="form-group">
            <label for="username">Username:</label>
            <input type="text" class="form-control" id="username" v-model="username">
          </div>
          <div class="form-group">
            <label for="password">Password:</label>
            <input type="password" class="form-control" id="password" v-model="password">
          </div>
          <button type="submit" class="btn btn-primary">Login</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      username: '',
      password: ''
    }
  },
  methods: {
    login () {
      if (this.username === 'admin' && this.password === '123456') {
        this.$emit('login')
      } else {
        alert('Invalid username or password.')
      }
    }
  }
}
</script>

<style>
.container {
  padding-top: 50px;
}
</style>
