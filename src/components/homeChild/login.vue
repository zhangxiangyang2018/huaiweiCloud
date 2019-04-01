<template>
  <el-row type="flex" justify="center">
    <el-form ref="loginForm" :model="user" label-width="80px">
      <el-form-item label="用户名">
        <el-input v-model="user.name" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item label="密码">
        <el-input v-model="user.pass" type="password" placeholder="请输入密码"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-upload" @click.enter="login">登录</el-button>
      </el-form-item>
    </el-form>
  </el-row>
</template>

<script>
import $ from 'jquery'
import axios from 'axios'
export default {
  data() {
    return {
      user: {
        name: '',
        pass: ''
      } // 用户信息存储
    }
  },
   $route () {
      console.log(this.$route.path)
    },
  mounted() {
     console.log(this.$route.path)
  },
  methods: {
    // 登陆
    login() {
     let that =this;
      let params = {
        userName: this.user.name,
        userPhone: this.user.pass
      }
      this.$httpPost('user/login', params).then(function(res) {
        if (res.data.code == '0') {
          that.$message({
            showClose: true,
            message: res.data.message,
            type: 'success',
            duration:1000
          })
          let   result = JSON.stringify(res.data.data)
           that.$store.dispatch("setUserId",res.data.data.userId)
            window.sessionStorage.setItem('userInfor',result)
             that.$router.push({path:'/home'})
        }
      
      })
    }
  }
}
</script>
