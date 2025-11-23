<template>
  <el-drawer v-model="visible" :title="title" direction="rtl" size="600" :before-close="cancel">
    <el-descriptions :column="2" border>
      <el-descriptions-item label="模块标题">{{ row.moduleTitle }}</el-descriptions-item>
      <el-descriptions-item label="业务类型">{{ row.businessType }}</el-descriptions-item>
      <el-descriptions-item label="请求地址">{{ row.reqUrl }}</el-descriptions-item>
      <el-descriptions-item label="主机地址">{{ row.host }}</el-descriptions-item>
      <el-descriptions-item label="操作地点">{{ row.location }}</el-descriptions-item>
      <el-descriptions-item label="操作人员">{{ row.operator }}</el-descriptions-item>
      <el-descriptions-item label="请求方法">{{ row.reqMethod }}</el-descriptions-item>
      <el-descriptions-item label="操作状态">
        <el-tag :type="row.status === 1 ? 'success' : 'danger'">
          {{ row.status === 1 ? '成功' : '失败' }}
        </el-tag>
      </el-descriptions-item>
      <el-descriptions-item label="操作时间" :span="2">{{ row.operTime }}</el-descriptions-item>
    </el-descriptions>
    <template #footer>
      <el-button @click="cancel">关闭</el-button>
    </template>
  </el-drawer>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const visible = ref(false)
const title = ref('')
const row = ref<any>({})

function acceptParams(params: any) {
  title.value = params.title
  row.value = params.row
  visible.value = true
}
function cancel() {
  visible.value = false
}

defineExpose({ acceptParams })
</script>
