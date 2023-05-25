<template>
  <div class="layout">
    <el-tabs type="border-card">
      <el-tab-pane label="密码登录">
        <el-form label-position="right" label-width="80px" style="max-width: 460px" class="loginForm">
          <el-form-item label="用户名：">
            <el-input v-model="username" />
          </el-form-item>
          <el-form-item label="密码：">
            <el-input type="password" v-model="password" />
          </el-form-item>

          <el-button
            type="primary"
            class="loginBtn"
            @click="loginPassword">登录
          </el-button>
          <el-button
            type="primary"
            class="loginBtn"
            @click="manage">信息管理页面
          </el-button>
        </el-form>
      </el-tab-pane>

      <el-tab-pane label="验证码登录">
        <el-form label-position="right" label-width="80px" style="max-width: 460px" class="loginForm">
          <el-form-item label="邮箱：">
            <el-input v-model="email" />
          </el-form-item>
          <el-form-item label="验证码：">
            <el-row>
              <el-input
                type="password"
                v-model="code"
                class="inpWidth"/>
              <el-button type="primary" @click="getIdentifyCode" plain style = "margin-left: 20px">
                获取验证码
              </el-button>
            </el-row>
          </el-form-item>
          <el-button
            type="primary"
            class="loginBtn"
            @click="loginCode">登录
          </el-button>
        </el-form>
      </el-tab-pane>

      <el-tab-pane label="usbKey登录">
        <el-form label-position="right" label-width="80px" style="max-width: 460px" class="loginForm">

        </el-form>
      </el-tab-pane>

      <el-tab-pane label="注册">
        <el-form
          label-position="right"
          label-width="100px"
          style="max-width: 460px"
          class="loginForm">
          <el-form-item label="邮箱：">
            <el-input v-model="rEmail" />
          </el-form-item>
          <el-form-item label="用户名：">
            <el-input v-model="rUsername" />
          </el-form-item>
          <el-form-item label="密码：">
            <el-input type="password" v-model="rPassword" />
          </el-form-item>
          <el-form-item label="确认密码：">
            <el-input
              type="password"
              v-model="confirmPassword"
              @blur="confirmFunc"/>
          </el-form-item>
          <el-form-item label="验证码：">
            <el-row>
              <el-input
                type="password"
                v-model="identifyCode"
                class="inpWidth"/>
              <el-button type="primary" @click="getIdentifyCode" plain>
                获取验证码
              </el-button>
            </el-row>
          </el-form-item>

          <el-button
            type="primary"
            class="loginBtn"
            @click="register">
            注册
          </el-button>
        </el-form>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script>
import { ref } from "vue";
import router from "@/router";
import { reactive, toRefs } from "@vue/reactivity";
import Axios from "axios";
import { ElMessage } from "element-plus";
export default {
  setup() {
    const form = reactive({
      username: "",
      password: "",
    });
    const codeForm = reactive({
      email: "",
      code: "",
    });
    const registerForm = reactive({
      rEmail: "",
      rPassword: "",
      rUsername: "",
      confirmPassword: "",
      identifyCode: "",
    });
    const confirm = ref(0)
    // 方法
    // 登录
    function loginPassword() {
      Axios.get("http://localhost:8081/key/login" , {
        params: {
          username: form.username,
          password: form.password
        }
      }).then(req =>{
        console.log(req.data)
        if(req.data == false){
          ElMessage.error("登录失败，用户名或密码错误")
        }
        else{
          ElMessage.success("登录成功")
        }
      })
    }

    //邮箱验证码登录
    function loginCode() {
      console.log(codeForm);
      Axios.get("http://localhost:8081/mail/checkTextMail",{
        params:{
          to: codeForm.email,
          code: codeForm.code
        }
      }).then(req =>{
        console.log(req.data)
        if(req.data == true){
          ElMessage.success("登录成功")
        }
        else{
          ElMessage.error("登录失败，验证码错误")
        }
      })
    }
    //信息管理页面
    function manage(){
      console.log("信息管理页面");
      router.push('/test')
    }
    // 注册
    function register() {
      console.log("注册", registerForm);
      if(confirm.value == 0){
        ElMessage.error("注册失败，密码与确认密码不一致")
      }
      else{
        Axios.get("http://localhost:8081/key/regist",{
          params:{
            username: registerForm.rUsername,
            password: registerForm.rPassword,
            email: registerForm.rEmail,
          }
        }).then(req =>{
          console.log(req.data)
          if(req.data == true){
            ElMessage.success("注册成功")
          }
          else{
            ElMessage.error("注册失败，账号或邮箱已使用")
          }
        })
      }
    }

    // 获取验证码
    function getIdentifyCode() {
      console.log("获取验证码");
      Axios.get("http://localhost:8081/mail/sendTextMail",{
        params:{
          to: registerForm.rEmail,
        }
      }).then(req =>{
        console.log(req.data)
        if(req.data == true){
          ElMessage.success("验证码已发送")
        }
        else{
          ElMessage.error("验证码发送失败")
        }
      })
    }

    // 确认密码
    const confirmFunc = () => {
      if (registerForm.confirmPassword !== registerForm.rPassword){
        ElMessage.error("密码与确认密码不一致.");
        confirm.value = 0
      }
      else confirm.value = 1;
    };
    
    return {
      ...toRefs(form),
      ...toRefs(codeForm),
      ...toRefs(registerForm),
      confirm,
      loginPassword,
      manage,
      loginCode,
      register,
      getIdentifyCode,
      confirmFunc,
    };
  },
};
</script>

<style scoped>
.layout {
  position: absolute;
  left: calc(50% - 200px);
  top: 20%;
  width: 400px;
}
.loginBtn {
  width: 100px;
}
.loginForm {
  text-align: center;
}
.checkBox {
  margin-left: 7px;
}
.inpWidth {
  width: 165px;
}
::-webkit-scrollbar {
  width: 0 !important;
}
::-webkit-scrollbar {
  width: 0 !important;height: 0;
}
</style>