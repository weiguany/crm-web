<template>
  <div class="table-box">
    <ProTable ref="proTable" title="合同列表" :columns="columns" :requestApi="ContractApi.page" :initParam="initParam" :dataCallback="dataCallback">
      <!-- 操作栏按钮 -->
      <template #operation="scope">
        <el-button type="success" link :icon="CircleCheckFilled" v-hasPermi="['sys:contract:pass']" @click="approvalContract(scope.row, 0)">审核通过</el-button>
        <el-button type="danger" link :icon="CircleCloseFilled" v-hasPermi="['sys:contract:reject']" @click="approvalContract(scope.row, 1)">审核不通过</el-button>
      </template>
    </ProTable>
    <el-dialog v-model="reasonDialog.visible" :title="reasonDialog.title" width="400px" @closed="reasonDialog.reason = ''">
      <el-input v-model="reasonDialog.reason" type="textarea" :rows="4" placeholder="请输入审核原因" maxlength="200" show-word-limit />
      <template #footer>
        <el-button @click="reasonDialog.visible = false">取消</el-button>
        <el-button type="primary" @click="submitApproval">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts" name="ContractManage">
import { ref, reactive } from 'vue'
import { ColumnProps } from '@/components/ProTable/interface'
import ProTable from '@/components/ProTable/index.vue'
import { ContractApi } from '@/api/modules/contract/index'
import { ContractStatusList } from '@/configs/enum'
import { CircleCheckFilled, CircleCloseFilled } from '@element-plus/icons-vue'
import { useHandleData } from '@/hooks/useHandleData'
import { ElMessage } from 'element-plus'

// 获取 ProTable 元素，调用其获取/刷新数据方法（还能获取到当前查询参数，方便导出携带参数）
const proTable = ref()

// 如果表格需要初始化请求参数，直接定义传给 ProTable（之后每次请求都会自动带上该参数，此参数更改之后也会一直带上，改变此参数会自动刷新表格数据）
const initParam = reactive({ status: 1 })

// dataCallback 是对于返回的表格数据做处理，如果后端返回的数据不是 dataList & total 这些字段，那么你可以在这里进行处理成这些字段
const dataCallback = (data: any) => {
  return {
    list: data.list,
    total: data.total
  }
}
// 表格配置项
const columns: ColumnProps[] = [
  {
    type: 'selection',
    fixed: 'left',
    width: 60
  },
  {
    prop: 'name',
    label: '合同名称',
    minWidth: 120,
    search: { el: 'input' }
  },
  {
    prop: 'number',
    label: '合同编号',
    minWidth: 120,
    search: { el: 'input' }
  },
  {
    // 推测此处可能为合同进度相关字段（截图中显示67%）
    prop: 'progress',
    label: '合同进度',
    minWidth: 100,
    search: { el: 'input' }
  },
  {
    prop: 'amount',
    label: '合同金额',
    minWidth: 100
  },
  {
    prop: 'receivedAmount',
    label: '已收到款项',
    minWidth: 140
  },
  {
    prop: 'status',
    label: '合同状态',
    minWidth: 120,
    enum: Object.values(ContractStatusList),
    search: { el: 'select' }
  },
  {
    prop: 'signTime',
    label: '签约时间',
    minWidth: 140
  },
  {
    prop: 'startTime',
    label: '合同开始时间',
    minWidth: 140
  },
  {
    prop: 'endTime',
    label: '合同结束时间',
    minWidth: 140
  },
  {
    prop: 'operation',
    label: '操作',
    fixed: 'right',
    width: 330
  }
]

//合同审核
// const approvalContract = async (row: any, type: number) => {
//   await useHandleData(ContractApi.approvalContract, { id: row.id, type }, type === 0 ? '合同审核通过' : '合同审核不通过')
//   proTable.value.getTableList()
// }

/* 弹窗状态 */
const reasonDialog = reactive({
  visible: false,
  title: '',
  reason: '',
  rowId: '',
  type: 0 // 0 通过  1 不通过
})

/* 打开弹窗 */
const approvalContract = (row: any, type: number) => {
  reasonDialog.rowId = row.id
  reasonDialog.type = type
  reasonDialog.title = type === 0 ? '审核通过原因' : '审核不通过原因'
  reasonDialog.visible = true
}

/* 提交审核 */
const submitApproval = async () => {
  if (!reasonDialog.reason.trim()) {
    ElMessage.warning('请输入审核原因')
    return
  }
  await useHandleData(
    ContractApi.approvalContract,
    { id: reasonDialog.rowId, type: reasonDialog.type, reason: reasonDialog.reason.trim() },
    reasonDialog.type === 0 ? '合同审核通过' : '合同审核不通过'
  )
  reasonDialog.visible = false
  proTable.value.getTableList()
}
</script>
