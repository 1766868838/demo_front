<template>
  <div class="login-container">
    <div class="login_box">
      <!-- 登录表单区域 -->
      <el-tabs :stretch="true">
        <el-tab-pane label="账号密码登录">
          <!-- 账号密码登录表单 -->
          <el-form ref="pwdLoginFormRef" :model="pwdLoginForm" :rules="pwdLoginFormRules">
            <!-- 用户名 -->
            <el-form-item prop="username">
              <el-input placeholder="用户名/邮箱/手机号" clearable prefix-icon="el-icon-user-solid
              " v-model="pwdLoginForm.username">
              </el-input>
            </el-form-item>
            <!-- 密码 -->
            <el-form-item prop="password">
              <el-input placeholder="密码" type="password" show-password prefix-icon="el-icon-lock"
                v-model="pwdLoginForm.password">
              </el-input>
            </el-form-item>
            <!-- 按钮区域 -->
            <el-form-item class="login_btns">
              <el-button type="primary" @click="pwdLogin" :loading="loading">登录</el-button>
              <el-button type="primary" @click='toRegister'>注册</el-button>
            </el-form-item>
          </el-form>
        </el-tab-pane>
        <el-tab-pane label="邮箱验证登录">
          <!-- 邮箱验证登录表单 -->
          <el-form ref="emailLoginFormRef" :model="emailLoginForm" :rules="emailLoginFormRules">
            <!-- 邮箱 -->
            <el-form-item prop="email">
              <el-input placeholder="邮箱" clearable prefix-icon="el-icon-message" v-model="emailLoginForm.email">
              </el-input>
            </el-form-item>
            <!-- 邮箱验证码 -->
            <el-form-item prop="emailCode">
              <el-input placeholder="验证码" prefix-icon="el-icon-key" v-model="emailLoginForm.emailCode">
                <template #append>
                  <el-button :disabled="disabled" @click="getEmailValidateCode">{{buttonText}}
                  </el-button>
                </template>
              </el-input>
            </el-form-item>
            <!-- 按钮区域 -->
            <el-form-item class="login_btns">
              <el-button type="primary" @click="emailLogin">登录</el-button>
              <el-button type="primary" @click='toRegister'>注册</el-button>
            </el-form-item>
          </el-form>
        </el-tab-pane>
        <!-- 使用usbKEY登录 -->
        <el-tab-pane label="使用usbKEY登录">
          <el-form>

          </el-form>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script>
import {
  ref,
  reactive,
  toRefs,
  getCurrentInstance
} from 'vue'
import qs from 'qs'
export default {
  setup () {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        // 合法的邮箱
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }
    const {
      proxy
    } = getCurrentInstance()
    const pwdLoginFormRef = ref(null)
    const emailLoginFormRef = ref(null)
    const state = reactive({
      pwdLoginForm: {
        username: 'admin',
        password: '123456'
      },
      emailLoginForm: {
        email: '',
        emailCode: ''
      },
      pwdLoginFormRules: {
        username: [{
          required: true,
          message: '请输入你的账号',
          trigger: 'blur'
        }],
        // 验证密码是否合法
        password: [{
          required: true,
          message: '请输入你的密码',
          trigger: 'blur'
        }]
      },
      emailLoginFormRules: {
        email: [{
          required: true,
          message: '请输入你的邮箱',
          trigger: 'blur'
        },
        {
          validator: checkEmail,
          trigger: 'blur'
        }
        ],
        emailCode: [{
          required: true,
          message: '请输入你获取到的验证码',
          trigger: 'blur'
        }]
      },
      loading: false,
      // 控制获取验证码
      buttonText: '获取验证码',
      disabled: false,
      duration: 60,
      timer: null
    })
    // 账号密码登录
    const pwdLogin = async () => {
      state.loading = true
      const obj = {
        username: state.pwdLoginForm.username,
        password: state.pwdLoginForm.password
      }
      try {
        const res = await new proxy.$request(proxy.$urls.m().pwdLogin, qs.stringify(obj)).modepost()
        console.log(res)
        if (res.data.success !== true) {
          new proxy.$tips(res.data.message, 'warning').mess_age()
        } else {
          new proxy.$tips(res.data.message, 'success').mess_age()
          localStorage.setItem('token', res.data.data.token)
          // 成功跳转页面
        }
        state.loading = false
      } catch (e) {
        state.loading = false
        new proxy.$tips('服务器发生错误', 'error').mess_age()
      }
    }
    // 获取邮箱验证码
    const getEmailValidateCode = () => {
      const obj = {
        email: state.emailLoginForm.email
      }
      axios.post('requestUrl', qs.stringify(obj)).then(
        res => {
          if (res.data.success !== true) {
            new proxy.$tips(res.data.message, 'warning').mess_age()
          } else {
            console.log(res.data)
            console.log(res.headers)
            new proxy.$tips(res.data.message, 'success').mess_age()
            // localStorage.setItem('token', res.data.data.token)
            state.timer = setInterval(() => {
              state.disabled = true
              const tmp = state.duration--
              state.buttonText = `${tmp}秒后重新获取`
              if (tmp <= 0) {
                clearInterval(state.timer)
                state.duration = 60
                state.buttonText = '重新获取验证码'
                state.disabled = false
              }
            }, 1000)
          }
        })
    }
    // 邮箱验证登录
    const emailLogin = async () => {
      state.loading = true
      const obj = {
        email: state.emailLoginForm.email,
        emailValidateCode: state.emailLoginForm.emailCode,
        token: localStorage.getItem('token')
      }
      try {
        const res = await new proxy.$request(proxy.$urls.m().emailLogin, qs.stringify(obj)).modepost()
        if (res.data.success !== true) {
          new proxy.$tips(res.data.message, 'warning').mess_age()
        } else {
          new proxy.$tips(res.data.message, 'success').mess_age()
          localStorage.setItem('token', res.data.data.token)
          // 成功跳转页面
        }
        state.loading = false
      } catch (e) {
        state.loading = false
        new proxy.$tips('服务器发生错误', 'error').mess_age()
      }
    }
    const toRegister = () => {
    }
    return {
      ...toRefs(state),
      pwdLoginFormRef,
      emailLoginFormRef,
      // phoneLoginFormRef,
      pwdLogin,
      getEmailValidateCode,
      emailLogin,
      // phoneLogin,
      toRegister
    }
  }
}
</script>
<style scoped>
  .login_container {
    width: 100%;
    height: 100%;
    position: fixed;
    background-size: 100% 100%;
  }

  .login_box {
    width: 465px;
    height: 435px;
    background-color: rgba(225, 225, 225, 0);
    border: 1px solid rgba(225, 225, 225, 0);
    box-shadow: 2px 3px 3px #aaaaaa;
    border-radius: 5px;
    position: relative;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }

  .login_btns {
    text-align: center;
  }

  .login_footer {
    position: absolute;
    left: 50%;
    top: 70%;
    transform: translate(-50%, -50%);
    border: 1px solid #ffffff;
    width: 395px;
    height: 30px;
    text-align: center;
  }
</style>

<style scoped>
  .login_box {
    backdrop-filter: blur(15px);
    box-shadow: 0 0 5px #fff;
  }

  .el-tabs >>> .el-tabs__header {
    padding: 5% 10% 0 10%;
  }

  .el-form {
    text-align: center;
  }

  .el-form-item>>>.el-form-item__error {
    left: 10%;
  }

  .el-input {
    width: 80%;
  }

</style>
