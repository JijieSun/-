<template>
  <header class="top-header">
    <div class="header-left">
      <div class="logo">🤖</div>
      <span class="system-name">智能陪伴机器人管理后台</span>
    </div>
    <div class="header-center" />
    <div class="header-right">
      <el-badge :value="3" class="header-item">
        <el-button text @click="notify">
          <el-icon><Bell /></el-icon>
          消息中心
        </el-button>
      </el-badge>
      <el-dropdown trigger="click" class="header-item">
        <el-button text>
          <el-icon><User /></el-icon>
          个人中心
        </el-button>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item>修改密码</el-dropdown-item>
            <el-dropdown-item>个人操作日志</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
      <el-dropdown trigger="click" class="header-item">
        <el-button text>
          <el-icon><Setting /></el-icon>
          全局系统配置
        </el-button>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item>数据加密规则</el-dropdown-item>
            <el-dropdown-item>存储监控</el-dropdown-item>
            <el-dropdown-item>日志开关</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
      <span class="account-info">演示管理员</span>
      <el-select
        v-model="permStore.role"
        class="role-switch"
        size="small"
        placeholder="权限切换"
        @change="onRoleChange"
      >
        <el-option
          v-for="r in roleOptions"
          :key="r.value"
          :label="r.label"
          :value="r.value"
        />
      </el-select>
      <el-button type="danger" link class="header-item" @click="logout">
        退出登录
      </el-button>
    </div>
  </header>
</template>

<script setup>
import { ElMessage } from 'element-plus'
import { usePermissionStore } from '@/stores/permission'
import { roleOptions } from '@/config/menus'
import { useRouter } from 'vue-router'

const permStore = usePermissionStore()
const router = useRouter()

function notify() {
  ElMessage.warning('3 条待处理：紧急呼救 1 · 待审核 2')
}

function onRoleChange() {
  ElMessage.info(`已切换为：${roleOptions.find((r) => r.value === permStore.role)?.label}`)
  router.push('/dashboard')
}

function logout() {
  ElMessage.success('已退出登录（演示）')
}
</script>

<style scoped>
.top-header {
  height: var(--admin-header-h);
  display: flex;
  align-items: center;
  padding: 0 20px;
  background: #fff;
  border-bottom: 1px solid #e8e8e8;
  position: sticky;
  top: 0;
  z-index: 100;
}
.header-left {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 280px;
}
.logo {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  background: linear-gradient(135deg, #1677ff, #69b1ff);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
}
.system-name {
  font-weight: 600;
  font-size: 16px;
  color: #1f1f1f;
  white-space: nowrap;
}
.header-center {
  flex: 1;
}
.header-right {
  display: flex;
  align-items: center;
  gap: 4px;
}
.role-switch {
  width: 130px;
  margin: 0 8px;
}
.account-info {
  font-size: 14px;
  color: #595959;
  padding: 0 8px;
  white-space: nowrap;
}
</style>
