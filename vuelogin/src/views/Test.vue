<template>
  <el-card>
    <el-row :gutter="20">
      <el-col :span="12">
        <!-- 搜索与添加区域 -->
          <el-input placeholder="请输入内容" >
          <template #append>
            <el-icon><Search /></el-icon>
          </template>
          </el-input>
      </el-col>
      <el-col :span="4">
        <el-button type="primary" @click="dialogFormVisible2 = true">添加用户</el-button>
      </el-col>
    </el-row>

    <!-- 用户列表区域-->
    <el-table :data="tests">
      <el-table-column type="index"></el-table-column>
      <el-table-column label="用户名" prop="username"></el-table-column>
      <el-table-column label="邮箱" prop="email"></el-table-column>
      <el-table-column fixed="right" label="Operations" width="120">
        <template #default="scope">
          <el-button link type="primary" size="small" @click="DeleteInf(scope.row)">Delete</el-button>
          <el-button link type="primary" size="small" @click="EditInf(scope.row)">Edit</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog v-model="dialogFormVisible" title="更新用户数据(留空代表该项数据不变)">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" autocomplete="off" />
        </el-form-item>
        <el-form-item label="密码" :label-width="formLabelWidth">
          <el-input v-model="form.password"  type="password" autocomplete="off" />
        </el-form-item>
        <el-form-item label="确认密码" :label-width="formLabelWidth">
          <el-input v-model="form.rPassword" type="password" autocomplete="off" @blur="confirmFunc"/>
        </el-form-item>
        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.email" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogFormVisible = false">Cancel</el-button>
          <el-button type="primary" @click="updateInf()">
            Confirm
          </el-button>
        </span>
      </template>
    </el-dialog>

    <el-dialog v-model="dialogFormVisible2" title="新增用户数据(留空代表该项数据不变)">
      <el-form :model="form">
        <el-form-item label="新用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" autocomplete="off" />
        </el-form-item>
        <el-form-item label="新密码" :label-width="formLabelWidth">
          <el-input v-model="form.password"  type="password" autocomplete="off" />
        </el-form-item>
        <el-form-item label="确认密码" :label-width="formLabelWidth">
          <el-input v-model="form.rPassword" type="password" autocomplete="off" @blur="confirmFunc"/>
        </el-form-item>
        <el-form-item label="新邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.email" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogFormVisible2 = false">Cancel</el-button>
          <el-button type="primary" @click="addInf()">
            Confirm
          </el-button>
        </span>
      </template>
    </el-dialog>

  </el-card>

</template>
  
<script setup>
  import { toRefs } from "@vue/reactivity";
  import axios from 'axios';
  import { ref } from "vue";
  import { ElMessage } from "element-plus";
  import { reactive } from 'vue';
  //数据
  const form = reactive({
    username: '',
    email: '',
    password: '',
    rPassword: '',
  });
  const formLabelWidth = '100px'
  let dialogFormVisible = ref(false)
  let dialogFormVisible2 = ref(false)
  let oldUsername = ref('')
  let tests = ref([])
  let total = 0
  let queryInfo = reactive({
    query: '',
    pagenum: 1,
    pagesize: 2
  });

  //初始调用
  getUserList()

  //方法
  function getUserList(){
    axios.get('http://localhost:8081/inf/findAll').then(function (res){
      //console.log(res.data)
      tests.value = res.data
      //console.log(tests.value)
      total = res.data.total
      
    }) 
  };

  //在跳出的对话框中点击确认后的界面
  function updateInf(){
    axios.get('http://localhost:8081/inf/update',{
      params:{
        newUsername : form.username,
        oldUsername : oldUsername.value,
        password: form.password,
        email: form.email
      }
    }).then(function(res){
      if(res.data == false){
        ElMessage.error("更新失败")
      }
      else{
        //隐藏对话框
        dialogFormVisible.value = false
        ElMessage.success("更新成功")
        getUserList()
      }
    })
  }

  function addInf(){
    
    axios.get("http://localhost:8081/key/regist",{
      params:{
        username: form.username,
        password: form.password,
        email: form.email,
      }
    }).then(req =>{
      console.log(req.data)
      if(req.data == true){
        ElMessage.success("注册成功")
        dialogFormVisible2.value = false;
        getUserList()
      }
      else{
        ElMessage.error("注册失败，账号或邮箱已使用")
      }
    })
  }

  const confirmFunc = () => {
    if (form.password !== form.rPassword)
      ElMessage.error("密码与确认密码不一致.");
  };

  function DeleteInf(row){
    axios.get('http://localhost:8081/inf/delete',{
      params:{
        username :row.username
      }
    }).then(function(res){
      if(res.data == false){
        ElMessage.error("删除失败")
    }
    else{
      ElMessage.success("删除成功")
        let list = tests.value.filter(val => val.username !== row.username);
        tests.value = list
      }
    })
  };

  function EditInf(row){
    dialogFormVisible.value = true
    oldUsername.value = row.username
    console.log(oldUsername.value)
  }

</script>
  
<style scoped>

.el-card{
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15) !important;
}
.el-table{
  margin-top: 15px;
  border: 1px;
}
::-webkit-scrollbar {
  width: 0 !important;
}
::-webkit-scrollbar {
  width: 0 !important;height: 0;
}
</style>

  
  