<template>
  <div class="page-card">
    <div class="page-head">
      <el-button text type="primary" @click="router.push('/users/memoir')">← 返回列表</el-button>
      <h2 class="page-title">{{ user?.userName }} · 回答精要</h2>
    </div>

    <template v-if="user">
      <el-descriptions :column="3" border size="small" class="user-meta">
        <el-descriptions-item label="手机">{{ user.phone }}</el-descriptions-item>
        <el-descriptions-item label="回忆录">{{ user.title }}</el-descriptions-item>
        <el-descriptions-item label="进度">
          {{ user.answeredCount }} / {{ MEMOIR_QUESTION_TOTAL }}
          （{{ progressPercent(user.answeredCount) }}%）
        </el-descriptions-item>
      </el-descriptions>

      <p class="page-desc">
        共 {{ MEMOIR_QUESTION_TOTAL }} 题，已答 <strong>{{ user.answeredCount }}</strong> 题。以下为各题
        <strong>回答内容精要</strong>（每题最多 {{ ANSWER_ESSENCE_MAX }} 字）；未答题显示「未回答」。
      </p>

      <div class="filter-bar">
        <el-form :inline="true">
          <el-form-item label="题号">
            <el-input v-model="questionNo" placeholder="如 1 或 1-50" clearable style="width: 140px" />
          </el-form-item>
          <el-form-item label="分类">
            <el-select v-model="category" placeholder="全部分类" clearable style="width: 130px">
              <el-option v-for="c in categories" :key="c" :label="c" :value="c" />
            </el-select>
          </el-form-item>
          <el-form-item label="状态">
            <el-select v-model="answerFilter" style="width: 120px">
              <el-option label="全部" value="all" />
              <el-option label="已回答" value="answered" />
              <el-option label="未回答" value="unanswered" />
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button @click="page = 1">查询</el-button>
          </el-form-item>
        </el-form>
      </div>

      <el-table :data="pagedRows" stripe border>
        <el-table-column prop="id" label="题号" width="72" align="center" />
        <el-table-column prop="category" label="分类" width="110" />
        <el-table-column prop="question" label="题目" min-width="200" show-overflow-tooltip />
        <el-table-column label="回答精要" min-width="320">
          <template #default="{ row }">
            <span v-if="row.essence" class="essence-text">{{ row.essence }}</span>
            <el-tag v-else type="info" size="small">未回答</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="字数" width="72" align="center">
          <template #default="{ row }">
            {{ row.essence ? row.essence.length : '—' }}
          </template>
        </el-table-column>
      </el-table>

      <div class="pagination-wrap">
        <el-pagination
          v-model:current-page="page"
          :page-size="pageSize"
          :total="filteredRows.length"
          layout="total, prev, pager, next, jumper"
        />
      </div>
    </template>

    <el-empty v-else description="用户不存在" />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import {
  MEMOIR_QUESTION_TOTAL,
  ANSWER_ESSENCE_MAX,
  memoirQuestionBank,
  getMemoirUserById,
  getUserAnswerEssence,
  progressPercent,
} from '@/data/memoirData'

const route = useRoute()
const router = useRouter()
const userId = computed(() => Number(route.params.userId))
const user = computed(() => getMemoirUserById(userId.value))

const questionNo = ref('')
const category = ref('')
const answerFilter = ref('all')
const page = ref(1)
const pageSize = 15

const categories = computed(() => [...new Set(memoirQuestionBank.map((q) => q.category))])

const allRows = computed(() => {
  if (!user.value) return []
  return memoirQuestionBank.map((q) => ({
    id: q.id,
    category: q.category,
    question: q.content,
    essence: getUserAnswerEssence(user.value, q),
  }))
})

const filteredRows = computed(() => {
  let rows = allRows.value
  if (category.value) rows = rows.filter((r) => r.category === category.value)
  if (answerFilter.value === 'answered') rows = rows.filter((r) => r.essence)
  if (answerFilter.value === 'unanswered') rows = rows.filter((r) => !r.essence)
  const qn = questionNo.value.trim()
  if (qn) {
    if (qn.includes('-')) {
      const [a, b] = qn.split('-').map((x) => parseInt(x.trim(), 10))
      if (!Number.isNaN(a) && !Number.isNaN(b)) {
        rows = rows.filter((r) => r.id >= a && r.id <= b)
      }
    } else {
      const n = parseInt(qn, 10)
      if (!Number.isNaN(n)) rows = rows.filter((r) => r.id === n)
    }
  }
  return rows
})

const pagedRows = computed(() => {
  const start = (page.value - 1) * pageSize
  return filteredRows.value.slice(start, start + pageSize)
})
</script>

<style scoped>
.page-head {
  margin-bottom: 12px;
}
.page-head .page-title {
  margin: 8px 0 0;
}
.user-meta {
  margin-bottom: 16px;
}
.page-desc {
  font-size: 13px;
  color: #64748b;
  line-height: 1.65;
  margin-bottom: 16px;
}
.essence-text {
  font-size: 13px;
  line-height: 1.6;
  color: #334155;
  display: -webkit-box;
  -webkit-line-clamp: 4;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.pagination-wrap {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}
</style>
