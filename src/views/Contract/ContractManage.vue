<template>
  <div class="table-box">
    <ProTable ref="proTable" title="合同列表" :columns="columns" :requestApi="ContractApi.page" :initParam="initParam" :dataCallback="dataCallback">
      <!-- 表格 header 按钮 -->
      <template #tableHeader>
        <el-button type="primary" :icon="CirclePlus" v-hasPermi="['sys:contract:add']" @click="openDrawer('新增')">新增合同</el-button>
      </template>
      <!-- 表格操作 -->
      <template #operation="scope">
        <el-button type="primary" link :icon="EditPen" v-hasPermi="['sys:contract:edit']" @click="openDrawer('编辑', scope.row)">编辑</el-button>
        <el-button type="success" link :icon="MessageBox" v-hasPermi="['sys:contract:print']" @click="openPrintDrawer('打印合同', scope.row)">打印</el-button>
        <el-button type="info" link :icon="Share" v-hasPermi="['sys:contract:audit']" v-if="scope.row.status === 0" @click="startApproval(scope.row)">审核</el-button>
      </template>
    </ProTable>
    <ContractDialog ref="dialogRef" />
    <PrintContractDialog ref="printDialogRef" />
  </div>
</template>

<script setup lang="ts" name="ContractManage">
import { ref, reactive } from 'vue'
import { ColumnProps } from '@/components/ProTable/interface'
import ProTable from '@/components/ProTable/index.vue'
import { ContractApi } from '@/api/modules/contract/index'
import { ContractStatusList } from '@/configs/enum'
import { CirclePlus, EditPen, MessageBox, Share } from '@element-plus/icons-vue'
import ContractDialog from './components/ContractDialog.vue'
import PrintContractDialog from './components/PrintContractDialog.vue'
import { useHandleData } from '@/hooks/useHandleData'

// 获取 ProTable 元素，调用其获取/刷新数据方法（还能获取到当前查询参数，方便导出携带参数）
const proTable = ref()

// 如果表格需要初始化请求参数，直接定义传给 ProTable（之后每次请求都会自动带上该参数，此参数更改之后也会一直带上，改变此参数会自动刷新表格数据）
const initParam = reactive({})

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
  },
  {
    prop: 'createByName',
    label: '创建人',
    minWidth: 100
  },
  {
    prop: 'createByEmail',
    label: '创建人邮箱',
    minWidth: 160
  }
]
// 打开 drawer(新增、查看、编辑)
const dialogRef = ref()

const openDrawer = (title: string, row: Partial<any> = {}) => {
  let params = {
    title,
    row: { ...row },
    isView: title === '查看',
    api: ContractApi.saveOrEdit,
    getTableList: proTable.value.getTableList,
    maxHeight: '550px'
  }
  dialogRef.value.acceptParams(params)
}

// 打印合同
const printDialogRef = ref()
const openPrintDrawer = (title: string, row: Partial<any> = {}) => {
  let params = {
    title,
    row: { ...row },
    isView: true,
    maxHeight: '600px',
    fullscreen: true
  }
  printDialogRef.value.acceptParams(params)
}
//开始审核合同
const startApproval = async (row: any) => {
  await useHandleData(ContractApi.startApproval, { id: row.id }, '发起合同审核')
  proTable.value.getTableList()
}
</script>
