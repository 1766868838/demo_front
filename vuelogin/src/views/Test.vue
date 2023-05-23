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
        <template #default="{row}">
          <el-button link type="primary" size="small" @click="DeleteInf(row)">Delete</el-button>
          <el-button link type="primary" size="small" @click="dialogFormVisible=true">Edit</el-button>
        </template>
      </el-table-column>
    </el-table>
  </el-card>


  <el-dialog v-model="dialogFormVisible" title="Shipping address">
    <el-form :model="form">
      <el-form-item label="Promotion name" :label-width="formLabelWidth">
        <el-input v-model="form.name" autocomplete="off" />
      </el-form-item>
      <el-form-item label="Zones" :label-width="formLabelWidth">
        <el-select v-model="form.region" placeholder="Please select a zone">
          <el-option label="Zone No.1" value="shanghai" />
          <el-option label="Zone No.2" value="beijing" />
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogFormVisible = false">Cancel</el-button>
        <el-button type="primary" @click="dialogFormVisible = false">
          Confirm
        </el-button>
      </span>
    </template>
  </el-dialog>
</template>
  
<script setup>
  import { toRefs } from "@vue/reactivity";
  import axios from 'axios';
  import { ref } from "vue";
  import { ElMessage } from "element-plus";
  import { reactive } from 'vue';
  const form = reactive({
    name: '',
    region: '',
    date1: '',
    date2: '',
    delivery: false,
    type: [],
    resource: '',
    desc: '',
  });
  const formLabelWidth = '140px'
  const dialogFormVisible = ref(false)
  let tests = ref([])
  let total = 0
  let queryInfo = reactive({
    query: '',
    pagenum: 1,
    pagesize: 2
  });
  getUserList()
  function getUserList(){
    axios.get('http://localhost:8081/inf/findAll').then(function (res){
      //console.log(res.data)
      tests.value = res.data
      //console.log(tests.value)
      total = res.data.total
      
    }) 
  };

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
    dialogFormVisible = true;
    console.log(dialogFormVisible)
  };

</script>
  
<style scoped>
.el-row{
  
}

.el-card{
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15) !important;
}
.el-table{
  margin-top: 15px;
  border: 1px;
}
</style>

  
  