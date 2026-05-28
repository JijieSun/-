<template>
  <div class="page-card">
    <h2 class="page-title">个人回忆录管理</h2>
    <p class="page-desc">
      基于问题库 <strong>{{ MEMOIR_QUESTION_TOTAL }}</strong> 道题采集进度；点击<strong>用户名</strong>查看各题回答精要（每题最多
      {{ ANSWER_ESSENCE_MAX }} 字）。
    </p>

    <div class="filter-bar">
      <el-form :inline="true">
        <el-form-item label="用户筛选">
          <el-input v-model="keyword" placeholder="手机号 / 用户名" clearable style="width: 220px" />
        </el-form-item>
        <el-form-item>
          <ActionButton label="查询" type="secondary" @click="search" />
        </el-form-item>
      </el-form>
    </div>

    <div class="toolbar-row">
      <ActionButton v-if="canUpdate" label="批量重新生成" type="primary" @click="toastAction('批量重新生成')" />
      <ActionButton v-if="canUpdate" label="归档已完成" type="secondary" @click="toastAction('归档')" />
      <ActionButton v-if="canExport" label="导出文件" type="export" @click="toastAction('导出')" />
      <span class="layout-hint">状态列展示答题进度</span>
    </div>

    <el-table :data="filteredUsers" stripe border>
      <el-table-column type="selection" width="48" />
      <el-table-column label="用户" min-width="100">
        <template #default="{ row }">
          <el-button type="primary" link @click="goAnswers(row)">{{ row.userName }}</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="phone" label="手机号" width="130" />
      <el-table-column prop="title" label="回忆录标题" min-width="160" show-overflow-tooltip />
      <el-table-column label="答题进度 / 状态" min-width="220">
        <template #default="{ row }">
          <div class="progress-cell">
            <div class="progress-text">
              {{ row.answeredCount }} / {{ MEMOIR_QUESTION_TOTAL }}
              <span class="pct">（{{ progressPercent(row.answeredCount) }}%）</span>
            </div>
            <el-progress
              :percentage="progressPercent(row.answeredCount)"
              :status="row.answeredCount >= MEMOIR_QUESTION_TOTAL ? 'success' : undefined"
              :stroke-width="8"
            />
            <el-tag size="small" :type="archiveTagType(row.archiveStatus)" class="archive-tag">
              {{ row.archiveStatus }}
            </el-tag>
          </div>
        </template>
      </el-table-column>
      <el-table-column prop="updatedAt" label="最近更新" width="160" />
      <el-table-column label="操作" width="140" fixed="right">
        <template #default="{ row }">
          <el-button type="primary" link size="small" @click="goAnswers(row)">查看回答</el-button>
          <el-button v-if="canUpdate" type="primary" link size="small" @click="toastAction('重新生成')">
            重生成
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-wrap">
      <el-pagination
        v-model:current-page="page"
        :page-size="10"
        :total="filteredUsers.length"
        layout="total, prev, pager, next"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import ActionButton from '@/components/ActionButton.vue'
import { usePermissionStore } from '@/stores/permission'
import {
  MEMOIR_QUESTION_TOTAL,
  ANSWER_ESSENCE_MAX,
  memoirUserRecords,
  progressPercent,
} from '@/data/memoirData'

const router = useRouter()
const permStore = usePermissionStore()
const keyword = ref('')
const page = ref(1)

const canUpdate = computed(() => permStore.can('update'))
const canExport = computed(() => permStore.can('export'))

const filteredUsers = computed(() => {
  const k = keyword.value.trim()
  if (!k) return memoirUserRecords
  return memoirUserRecords.filter(
    (u) => u.userName.includes(k) || u.phone.includes(k)
  )
})

function archiveTagType(status) {
  const map = { 已完成: 'success', 采集中: '', 已暂停: 'info' }
  return map[status] || 'info'
}

function search() {
  ElMessage.info(`共 ${filteredUsers.value.length} 条`)
}

function goAnswers(row) {
  router.push({ name: 'memoir-user-answers', params: { userId: row.id } })
}

function toastAction(label) {
  ElMessage.success(`[演示] ${label}`)
}
</script>

<style scoped>
.page-desc {
  margin: -8px 0 16px;
  font-size: 13px;
  color: #64748b;
  line-height: 1.65;
}
.toolbar-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 12px;
  align-items: center;
}
.layout-hint {
  margin-left: auto;
  font-size: 12px;
  color: #8c8c8c;
}
.progress-cell {
  padding: 4px 0;
}
.progress-text {
  font-size: 13px;
  margin-bottom: 6px;
}
.pct {
  color: #1677ff;
  font-weight: 600;
}
.archive-tag {
  margin-top: 6px;
}
.pagination-wrap {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}
</style>
