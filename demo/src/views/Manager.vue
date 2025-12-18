<template>
  <div class="admin-container">
    <!-- 顶部导航栏 -->
    <el-header class="header">
      <div class="header-left">
        <el-button
          icon="Menu"
          size="small"
          class="menu-toggle"
          @click="isCollapse = !isCollapse"
        />
        <div class="logo">
          <img src="/src/assets/img/logo.png" alt="Logo" class="logo-img" />
          <span class="logo-title">bug管理系统</span>
        </div>
      </div>
      <div class="header-right">
        <el-dropdown>
          <span class="user-info">
            <el-avatar icon="User" class="avatar" />
            <span class="username">{{ data.user.name }}</span>
            <el-icon class="arrow">
              <ArrowDownBold />
            </el-icon>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item @click="router.push('/manager/about')"
                >个人中心</el-dropdown-item
              >
              <el-dropdown-item>退出登录</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </div>
    </el-header>

    <div class="main-content">
      <!-- 侧边导航菜单 -->
      <el-aside
        class="sidebar"
        :width="isCollapse ? '64px' : '200px'"
        :style="{ transition: 'width 0.3s ease' }"
      >
        <el-menu
          :default-active="router.currentRoute.value.path"
          :collapse="isCollapse"
          :collapse-transition="false"
          background-color="#0f172a"
          text-color="#e2e8f0"
          active-text-color="#38bdf8"
          router
        >
          <el-menu-item index="/manager/home">
            <el-icon>
              <HomeFilled />
            </el-icon>
            <template #title>主页</template>
          </el-menu-item>
          <el-menu-item index="/manager/about">
            <el-icon>
              <User />
            </el-icon>
            <template #title>帮助</template>
          </el-menu-item>
          <el-menu-item index="/manager/user">
            <el-icon>
              <PieChart />
            </el-icon>
            <template #title>用户表</template>
          </el-menu-item>
          <el-menu-item index="table">
            <el-icon>
              <PieChart />
            </el-icon>
            <template #title>数据表格</template>
          </el-menu-item>
          <el-menu-item index="form">
            <el-icon>
              <Document />
            </el-icon>
            <template #title>表单管理</template>
          </el-menu-item>
          <el-menu-item index="notice">
            <el-icon>
              <Bell />
            </el-icon>
            <template #title>公告管理</template>
          </el-menu-item>
        </el-menu>
      </el-aside>

      <!-- 主内容区域 -->
      <el-main class="main">
        <router-view />
      </el-main>
    </div>
  </div>
</template>

<script setup>
import router from "@/router/index.js";
import "@/assets/css/manager.css";
import { ref, reactive } from "vue";

const data = reactive({
  user: JSON.parse(localStorage.getItem("xm-pro-user")),
});

// 控制菜单折叠状态
const isCollapse = ref(false);
</script>

<style scoped></style>
