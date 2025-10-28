<template>
  <div v-cloak>
    <el-menu
      :default-active="currentRoute"
      mode="horizontal"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b"
      router>
      <el-menu-item index="/task">任务管理</el-menu-item>
      <el-menu-item index="/host">任务节点</el-menu-item>
      <el-menu-item v-if="userStore.isAdmin" index="/user">用户管理</el-menu-item>
      <el-menu-item v-if="userStore.isAdmin" index="/system">系统管理</el-menu-item>
      <el-sub-menu v-if="userStore.isLogin" index="user-menu" style="margin-left: auto;">
        <template #title>
          <el-icon><User /></el-icon>
          <span>{{ userStore.username || '用户' }}</span>
        </template>
        <el-menu-item @click="$router.push('/user/edit-my-password')">修改密码</el-menu-item>
        <el-menu-item @click="$router.push('/user/two-factor')">双因素认证</el-menu-item>
        <el-menu-item @click="logout">退出</el-menu-item>
      </el-sub-menu>
    </el-menu>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useUserStore } from '../../stores/user'
import { ArrowDown, User } from '@element-plus/icons-vue'

const route = useRoute()
const router = useRouter()
const userStore = useUserStore()

const currentRoute = computed(() => {
  if (route.path === '/') return '/task'
  const segments = route.path.split('/')
  return `/${segments[1]}`
})

const logout = () => {
  userStore.logout()
  router.push('/user/login').then(() => {
    window.location.reload()
  })
}
</script>

<style scoped>
.el-menu {
  display: flex;
  align-items: center;
}
</style>