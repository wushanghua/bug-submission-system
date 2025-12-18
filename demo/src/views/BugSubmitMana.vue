<template>
  <div class="app-container">
    <!-- 横向导航栏 -->
    <el-header class="header">
      <div class="logo">
        <span class="iconfont-logo">🐞</span>
        <span class="logo-text">Bug提交系统</span>
      </div>
      <!-- 横向导航菜单 -->
      <el-menu
        mode="horizontal"
        :default-active="router.currentRoute.value.path"
        router
        background-color="rgb(17, 17,17)"
        text-color="#fff"
        :ellipsis="false"
        active-text-color="#ffd04b"
      >
        <el-menu-item index="/bugmana/userhome"> <el-icon><House /></el-icon>首页</el-menu-item>
        <el-menu-item index="/bugmana/submit_bug"><el-icon><CirclePlus /></el-icon>提交Bug</el-menu-item>
        <el-menu-item index="/bugmana/bug_list"><el-icon><Document /></el-icon>bug列表</el-menu-item>

        <!-- 已登录：显示下拉菜单 -->
        <el-sub-menu index="loginMenu" >
            <template #title>  <img :src="data.userPic"
                                    style="height: 50px; width: 50px; border-radius: 50%; ">{{data.username}}</template>

           
          <el-menu-item index="/bugmana/userinfo">个人中心</el-menu-item>

          <el-menu-item @click="exit" index="">退出登录</el-menu-item>
        </el-sub-menu>
      </el-menu>
    </el-header>

    <!-- 路由内容区（路由跳转后显示的内容） -->
    <el-main class="main-content">
      <router-view />
    </el-main>
  </div>
</template>

<script setup>
import {ref, reactive, watch, onMounted} from "vue";
import { ElMessage } from "element-plus";
import router from "@/router/index.js";
import { jwtDecode } from 'jwt-decode';
import axios from "@/utils/axios.js";
// 从 localStorage 中获取用户信息
const data = reactive({
  token:localStorage.getItem('xm-pro-user'),
  username: "",
  userPic:""
});

onMounted(()=>{
  axios.get("/user/userInfo",{
    headers: {
      'Authorization': data.token
    }
  }).then(res=>{
    if (res.code ===0){

      data.userPic =res.data.userPic
      data.username =res.data.username
    }else {
      console.error(res.msg)
    }
  })

})



// 退出登录
const exit = () => {
  localStorage.removeItem("xm-pro-user");
  data.user = null;
  ElMessage.success("已退出登录");
  location.href = "/login";
};
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  padding: 0 20px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  background-color: rgb(17, 17, 17);
  z-index: 10;
}

.logo {
  display: flex;
  align-items: center;
  gap: 10px;
}

.iconfont-logo {
  font-size: 24px;
}

.logo-text {
  font-size: 18px;
  font-weight: 600;
  color: white;
}

.main-content {
  flex: 1;
  padding: 20px;
  overflow: auto;
  max-width: 80%;
  margin-left: 15%;
  margin-right: 15%;
  background-color: rgb(216, 215, 215);
}

横向菜单样式调整 :deep(.el-menu--horizontal) {
  height: 60px;
  line-height: 60px;
}

:deep(.el-menu-item) {
  height: 60px;
  line-height: 60px;
  padding: 0 20px;
}
</style>
