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
        <el-button type="primary">添加用户</el-button>
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
          <el-button link type="primary" size="small" @click="EditInf(scope.$index)">Edit</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog v-model="dialogFormVisible" title="更新用户数据(留空代表该项数据不变)">
      <el-form :model="form">
        <el-form-item label="新用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" autocomplete="off" />
        </el-form-item>
        <el-form-item label="新密码" :label-width="formLabelWidth">
          <el-input v-model="form.password" autocomplete="off" />
        </el-form-item>
        <el-form-item label="新邮箱" :label-width="formLabelWidth">
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
  });
  const formLabelWidth = '100px'
  let dialogFormVisible = ref(false)
  let nowIndex = ref(0)
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
  function updateInf(row){
    console.log(row)
    axios.get('http://localhost:8081/inf/update',{
      username : form.username,
      password: form.password,
      email: form.email
    }).then(function(res){
      if(res.data == false){
        ElMessage.error("更新失败")
      }
      else{
        //隐藏对话框
        dialogFormVisible.value = false
      }
    })
  }

  function DeleteInf(row){
    console.log(row.value)
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
    nowIndex = row
    console.log(row)
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

  
  