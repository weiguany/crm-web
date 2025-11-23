<template>
  <div class="table-box">
    <ProTable
      ref="proTable"
      title="公海管理"
      :columns="columns"
      :requestApi="CustomerApi.page"
      :initParam="initParam"
      :dataCallback="dataCallback"
      :searchCol="{ xs: 2, sm: 3, md: 4, lg: 6, xl: 8 }"
    >
      <!-- 表格操作列 -->
      <template #operation="scope">
        <el-button type="primary" link :icon="EditPen" v-hasPermi="['sys:customer:private']" @click="customerToPrivate(scope.row.id)"> 客户领取 </el-button>
      </template>
    </ProTable>
  </div>
</template>

<script setup lang="ts" name="PublicManage">
import { ref, reactive } from 'vue'
import { ColumnProps } from '@/components/ProTable/interface'
import ProTable from '@/components/ProTable/index.vue'
import { CustomerApi } from '@/api/modules/customer'
import { GenderList, IsKeyDecisionMakerList } from '@/configs/enum'
import { EditPen } from '@element-plus/icons-vue'
import { useHandleData } from '@/hooks/useHandleData'

// ProTable 实例，用于调用表格方法
const proTable = ref<InstanceType<typeof ProTable>>()

// 初始请求参数（公海标识 isPublic: 1）
const initParam = reactive({ isPublic: 1 })
const dataSize = ref(0)

// 数据格式化回调：适配 ProTable 所需的 list/total 结构
const dataCallback = (data: any) => {
  dataSize.value = data.list.size
  return {
    list: data.list,
    total: data.total
  }
}

// 表格列配置
const columns: ColumnProps[] = [
  { type: 'selection', fixed: 'left', width: 60 },
  {
    prop: 'name',
    label: '客户名称',
    minWidth: 120,
    search: { el: 'input' }
  },
  {
    prop: 'phone',
    label: '手机号',
    minWidth: 160,
    search: { el: 'input' }
  },
  {
    prop: 'email',
    label: '邮箱',
    minWidth: 120
  },
  {
    prop: 'gender',
    label: '性别',
    enum: Object.values(GenderList),
    minWidth: 120
  },
  {
    prop: 'isKeyDecisionMaker',
    label: '是否为关键决策人',
    enum: Object.values(IsKeyDecisionMakerList),
    minWidth: 140
  },
  { prop: 'operation', label: '操作', fixed: 'right', width: 130 }
]

// 领取客户逻辑：调用接口 + 刷新表格
const customerToPrivate = async (id: any) => {
  await useHandleData(CustomerApi.toPrivate, { id: id }, '领取客户')
  proTable.value?.clearSelection()
  proTable.value?.getTableList()
}
</script>
