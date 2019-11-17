<template>
  <div class="site-wrapper site-page--login">
    <div class="login-main">
      <h3 class="login-title">管理员登录</h3>
      <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
               status-icon>
        <el-form-item prop="userName">
          <el-input v-model="dataForm.userName" placeholder="帐号"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="dataForm.password" type="password" placeholder="密码"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button class="login-btn-submit" type="primary" @click="dataFormSubmit()">登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
    import aes from '@/utils/encrypte.js'
    // import aes from  'crypto-js'

    export default {
        data() {
            return {
                dataForm: {
                    userName: '',
                    password: '',
                    uuid: '',
                    // captcha: ''
                },
                dataRule: {
                    userName: [{
                        required: true,
                        message: '帐号不能为空',
                        trigger: 'blur'
                    }],
                    password: [{
                        required: true,
                        message: '密码不能为空',
                        trigger: 'blur'
                    }],
                    captcha: [{
                        required: true,
                        message: '验证码不能为空',
                        trigger: 'blur'
                    }]
                },
                captchaPath: ''
            }
        },
        created() {},
        methods: {
            // 提交表单
            dataFormSubmit() {
                this.$refs['dataForm'].validate((valid) => {
                    if (valid) {
                        this.$http({
                            url: this.$http.adornUrl('/sys/login'),
                            method: 'post',
                            data: this.$http.adornData({
                                'username': aes.Encrypt(JSON.stringify(this.dataForm.userName)),
                                'password': aes.Encrypt(JSON.stringify(this.dataForm.password))
                            })
                        }).then(({
                            data
                        }) => {
                            console.log("unknown")
                            if (data && data.code === 0) {
                                this.$cookie.set('token', data.data.token)
                                this.$router.replace({
                                    name: 'home'
                                })
                            } else {
                                this.$message.error(data.data.msg)
                            }
                        })
                    }
                })
            }
        }
    }
</script>

<style lang="scss">
    .site-wrapper.site-page--login {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        background-image: url(~@/assets/img/login_bg.jpg);
        background-size: cover;
        z-index: -1;
        overflow: hidden;
        .login-main {
            margin: 20vh auto;
            padding: 50px 80px;
            width: 500px;
            background-color: #fff;
        }
        .login-title {
            font-size: 16px;
        }
        .login-captcha {
            overflow: hidden;
            >img {
                width: 100%;
                cursor: pointer;
            }
        }
        .login-btn-submit {
            width: 100%;
            margin-top: 38px;
        }
    }
</style>