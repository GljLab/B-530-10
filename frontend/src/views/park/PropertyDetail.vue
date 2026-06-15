<template>
  <div class="property-detail">
    <div class="detail-header">
      <el-button @click="handleBack">
        <el-icon><ArrowLeft /></el-icon>返回
      </el-button>
    </div>

    <div v-loading="loading" class="detail-content">
      <div class="property-header">
        <div class="property-title">
          <h1>{{ property.propertyCode }}</h1>
          <el-tag :type="getStatusTagType(property.status)" size="large">
            {{ getStatusLabel(property.status) }}
          </el-tag>
        </div>
        <div class="property-actions">
          <el-button v-if="hasEditPermission" type="primary" @click="handleEdit">
            <el-icon><Edit /></el-icon>编辑房产
          </el-button>
          <el-button v-if="hasChangeStatusPermission" type="warning" @click="handleChangeStatus">
            <el-icon><Promotion /></el-icon>修改状态
          </el-button>
          <el-button v-if="hasDeletePermission" type="danger" @click="handleDelete">
            <el-icon><Delete /></el-icon>删除房产
          </el-button>
        </div>
      </div>

      <el-row :gutter="20">
        <el-col :span="12">
          <el-card class="info-card">
            <template #header>
              <span class="card-title">基础信息</span>
            </template>
            <el-descriptions :column="2" border>
              <el-descriptions-item label="楼宇名称">{{ property.buildingName || '-' }}</el-descriptions-item>
              <el-descriptions-item label="所在楼层">{{ property.floorNumber || '-' }}</el-descriptions-item>
              <el-descriptions-item label="房产类型">
                {{ getPropertyTypeLabel(property.propertyType, property.propertySubType) }}
              </el-descriptions-item>
              <el-descriptions-item label="房产状态">
                <el-tag :type="getStatusTagType(property.status)" size="small">
                  {{ getStatusLabel(property.status) }}
                </el-tag>
              </el-descriptions-item>
              <el-descriptions-item label="建筑面积">{{ property.buildingArea || '-' }} ㎡</el-descriptions-item>
              <el-descriptions-item label="套内面积">{{ property.insideArea || '-' }} ㎡</el-descriptions-item>
              <el-descriptions-item label="公摊面积">{{ property.sharedArea || '-' }} ㎡</el-descriptions-item>
              <el-descriptions-item label="户型">{{ property.houseType || '-' }}</el-descriptions-item>
              <el-descriptions-item label="朝向">{{ property.orientation || '-' }}</el-descriptions-item>
              <el-descriptions-item label="装修状况">{{ property.decorationStatus || '-' }}</el-descriptions-item>
              <el-descriptions-item label="特殊标识">
                <div v-if="property.specialTagList && property.specialTagList.length > 0">
                  <el-tag v-for="tag in property.specialTagList" :key="tag" size="small" style="margin-right: 5px;">
                    {{ tag }}
                  </el-tag>
                </div>
                <span v-else>-</span>
              </el-descriptions-item>
            </el-descriptions>
          </el-card>
        </el-col>

        <el-col :span="12">
          <el-card class="info-card">
            <template #header>
              <span class="card-title">产权信息</span>
            </template>
            <el-descriptions :column="2" border>
              <el-descriptions-item label="产权性质">{{ property.propertyRightType || '-' }}</el-descriptions-item>
              <el-descriptions-item label="产权年限">
                {{ property.propertyRightYears ? property.propertyRightYears + '年' : '-' }}
              </el-descriptions-item>
              <el-descriptions-item label="产权证号" :span="2">
                {{ property.propertyCertNo || '-' }}
              </el-descriptions-item>
              <el-descriptions-item label="交付日期" :span="2">
                {{ property.deliveryDate || '-' }}
              </el-descriptions-item>
            </el-descriptions>
          </el-card>
        </el-col>
      </el-row>

      <el-row :gutter="20" style="margin-top: 20px;">
        <el-col :span="12">
          <el-card class="info-card">
            <template #header>
              <span class="card-title">使用情况</span>
            </template>
            <el-descriptions :column="2" border>
              <el-descriptions-item label="当前状态">
                <el-tag :type="getStatusTagType(property.status)" size="small">
                  {{ getStatusLabel(property.status) }}
                </el-tag>
              </el-descriptions-item>
              <el-descriptions-item label="产权人">
                <span style="color: #909399;">待关联</span>
              </el-descriptions-item>
              <el-descriptions-item label="租户信息">
                <span v-if="property.status === 4" style="color: #909399;">待关联</span>
                <span v-else>-</span>
              </el-descriptions-item>
            </el-descriptions>
          </el-card>
        </el-col>

        <el-col :span="12">
          <el-card class="info-card">
            <template #header>
              <span class="card-title">备注信息</span>
            </template>
            <div class="remark-content">
              {{ property.remark || '暂无备注' }}
            </div>
          </el-card>
        </el-col>
      </el-row>

      <el-card class="info-card" style="margin-top: 20px;">
        <template #header>
          <span class="card-title">状态变更历史</span>
        </template>
        <el-timeline>
          <el-timeline-item
            v-for="(log, index) in statusLogs"
            :key="log.id"
            :timestamp="log.createTime"
            :type="index === 0 ? 'primary' : ''"
            placement="top">
            <el-card shadow="never" class="log-card">
              <h4>{{ getStatusLabel(log.newStatus) }}</h4>
              <p v-if="log.oldStatus">
                由 <el-tag size="small" :type="getStatusTagType(log.oldStatus)">{{ getStatusLabel(log.oldStatus) }}</el-tag> 变更
              </p>
              <p v-else>初始状态</p>
              <p v-if="log.changeReason" class="reason">变更原因：{{ log.changeReason }}</p>
              <p class="operator">操作人：{{ log.operatorName || '系统' }}</p>
            </el-card>
          </el-timeline-item>
        </el-timeline>
        <div v-if="statusLogs.length === 0" class="empty-tip">
          暂无状态变更记录
        </div>
      </el-card>
    </div>

    <el-dialog v-model="editVisible" title="编辑房产" width="700px" @close="handleEditClose">
      <el-form ref="editFormRef" :model="editForm" :rules="editRules" label-width="100px">
        <el-tabs v-model="editActiveTab">
          <el-tab-pane label="房产属性" name="attr">
            <el-row :gutter="20">
              <el-col :span="8">
                <el-form-item label="建筑面积" prop="buildingArea">
                  <el-input-number v-model="editForm.buildingArea" :min="0" :precision="2" style="width: 100%;" />
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="套内面积" prop="insideArea">
                  <el-input-number v-model="editForm.insideArea" :min="0" :precision="2" style="width: 100%;" />
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="公摊面积" prop="sharedArea">
                  <el-input-number v-model="editForm.sharedArea" :min="0" :precision="2" style="width: 100%;" />
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="8">
                <el-form-item label="户型" prop="houseType">
                  <el-select v-model="editForm.houseType" placeholder="请选择" clearable style="width: 100%;">
                    <el-option label="一室" value="一室" />
                    <el-option label="两室" value="两室" />
                    <el-option label="三室" value="三室" />
                    <el-option label="四室及以上" value="四室及以上" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="朝向" prop="orientation">
                  <el-select v-model="editForm.orientation" placeholder="请选择" clearable style="width: 100%;">
                    <el-option label="东" value="东" />
                    <el-option label="南" value="南" />
                    <el-option label="西" value="西" />
                    <el-option label="北" value="北" />
                    <el-option label="东南" value="东南" />
                    <el-option label="西南" value="西南" />
                    <el-option label="东北" value="东北" />
                    <el-option label="西北" value="西北" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="装修状况" prop="decorationStatus">
                  <el-select v-model="editForm.decorationStatus" placeholder="请选择" clearable style="width: 100%;">
                    <el-option label="毛坯" value="毛坯" />
                    <el-option label="简装" value="简装" />
                    <el-option label="精装" value="精装" />
                    <el-option label="豪装" value="豪装" />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
          </el-tab-pane>
          <el-tab-pane label="产权信息" name="right">
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="产权性质" prop="propertyRightType">
                  <el-select v-model="editForm.propertyRightType" placeholder="请选择" clearable style="width: 100%;">
                    <el-option label="商品房" value="商品房" />
                    <el-option label="经济适用房" value="经济适用房" />
                    <el-option label="公租房" value="公租房" />
                    <el-option label="商铺产权" value="商铺产权" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="产权年限" prop="propertyRightYears">
                  <el-select v-model="editForm.propertyRightYears" placeholder="请选择" clearable style="width: 100%;">
                    <el-option label="40年" :value="40" />
                    <el-option label="50年" :value="50" />
                    <el-option label="70年" :value="70" />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="产权证号" prop="propertyCertNo">
                  <el-input v-model="editForm.propertyCertNo" :disabled="!isManager" placeholder="请输入产权证号" />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="交付日期" prop="deliveryDate">
                  <el-date-picker v-model="editForm.deliveryDate" type="date" placeholder="选择日期" style="width: 100%;" />
                </el-form-item>
              </el-col>
            </el-row>
          </el-tab-pane>
          <el-tab-pane label="其他信息" name="other">
            <el-form-item label="特殊标识">
              <el-checkbox-group v-model="editForm.specialTagList">
                <el-checkbox label="学区房">学区房</el-checkbox>
                <el-checkbox label="人才房">人才房</el-checkbox>
                <el-checkbox label="拆迁房">拆迁房</el-checkbox>
              </el-checkbox-group>
            </el-form-item>
            <el-form-item label="备注">
              <el-input v-model="editForm.remark" type="textarea" :rows="4" placeholder="请输入备注" />
            </el-form-item>
          </el-tab-pane>
        </el-tabs>
      </el-form>
      <template #footer>
        <el-button @click="editVisible = false">取消</el-button>
        <el-button type="primary" @click="handleEditSubmit">确定</el-button>
      </template>
    </el-dialog>

    <el-dialog v-model="statusDialogVisible" title="修改状态" width="400px">
      <el-form label-width="80px">
        <el-form-item label="当前状态">
          <el-tag :type="getStatusTagType(property.status)">{{ getStatusLabel(property.status) }}</el-tag>
        </el-form-item>
        <el-form-item label="新状态">
          <el-select v-model="newStatus" placeholder="请选择" style="width: 100%;">
            <el-option label="待售" :value="1" />
            <el-option label="已售未交付" :value="2" />
            <el-option label="业主自住" :value="3" />
            <el-option label="业主出租" :value="4" />
            <el-option label="空置" :value="5" />
            <el-option label="装修中" :value="6" />
            <el-option label="查封" :value="7" />
          </el-select>
        </el-form-item>
        <el-form-item label="变更原因">
          <el-input v-model="changeReason" type="textarea" :rows="3" placeholder="请输入变更原因（可选）" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="statusDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="confirmChangeStatus">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { ElMessage, ElMessageBox } from 'element-plus'
import { ArrowLeft, Edit, Delete, Promotion } from '@element-plus/icons-vue'
import api from '@/api'

const route = useRoute()
const router = useRouter()
const userStore = useUserStore()

const loading = ref(false)
const property = ref({})
const statusLogs = ref([])

const editVisible = ref(false)
const editActiveTab = ref('attr')
const editFormRef = ref(null)
const editForm = reactive({
  id: null,
  buildingArea: null,
  insideArea: null,
  sharedArea: null,
  houseType: '',
  orientation: '',
  decorationStatus: '',
  propertyRightType: '',
  propertyRightYears: null,
  propertyCertNo: '',
  deliveryDate: null,
  specialTagList: [],
  remark: ''
})

const editRules = {}

const statusDialogVisible = ref(false)
const newStatus = ref(null)
const changeReason = ref('')

const hasEditPermission = computed(() => userStore.hasPermission('park:property:edit'))
const hasDeletePermission = computed(() => userStore.hasPermission('park:property:delete'))
const hasChangeStatusPermission = computed(() => userStore.hasPermission('park:property:changeStatus'))
const isManager = computed(() => userStore.hasPermission('park:property:delete'))

onMounted(() => {
  loadDetail()
})

function loadDetail() {
  const id = route.params.id
  if (!id) return

  loading.value = true
  api.parkProperty.getById(id).then(res => {
    property.value = res.data || {}
    statusLogs.value = res.data.statusLogs || []
  }).finally(() => {
    loading.value = false
  })
}

function handleBack() {
  router.back()
}

function getPropertyTypeLabel(type, subType) {
  if (subType) return subType
  const map = { 1: '住宅', 2: '商业', 3: '其他' }
  return map[type] || '-'
}

function getStatusLabel(status) {
  const map = {
    1: '待售',
    2: '已售未交付',
    3: '业主自住',
    4: '业主出租',
    5: '空置',
    6: '装修中',
    7: '查封'
  }
  return map[status] || '-'
}

function getStatusTagType(status) {
  const map = {
    1: 'warning',
    2: 'info',
    3: 'success',
    4: '',
    5: 'info',
    6: 'warning',
    7: 'danger'
  }
  return map[status] || 'info'
}

function handleEdit() {
  Object.assign(editForm, property.value)
  editForm.specialTagList = property.value.specialTagList || []
  editActiveTab.value = 'attr'
  editVisible.value = true
}

function handleEditClose() {
  editFormRef.value?.resetFields()
}

function handleEditSubmit() {
  const submitData = { ...editForm }
  if (submitData.specialTagList && submitData.specialTagList.length > 0) {
    submitData.specialTags = submitData.specialTagList.join(',')
  }

  api.parkProperty.update(submitData).then(() => {
    ElMessage.success('修改成功')
    editVisible.value = false
    loadDetail()
  })
}

function handleDelete() {
  ElMessageBox.confirm(`确定要删除房产 "${property.value.propertyCode}" 吗？`, '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    api.parkProperty.delete(property.value.id).then(() => {
      ElMessage.success('删除成功')
      router.back()
    })
  }).catch(() => {})
}

function handleChangeStatus() {
  newStatus.value = null
  changeReason.value = ''
  statusDialogVisible.value = true
}

function confirmChangeStatus() {
  if (!newStatus.value) {
    ElMessage.warning('请选择新状态')
    return
  }
  api.parkProperty.changeStatus(property.value.id, newStatus.value, changeReason.value).then(() => {
    ElMessage.success('状态修改成功')
    statusDialogVisible.value = false
    loadDetail()
  })
}
</script>

<style scoped lang="scss">
.property-detail {
  padding: 20px;
}

.detail-header {
  margin-bottom: 20px;
}

.detail-content {
  .property-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    padding: 20px;
    background: #fff;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);

    .property-title {
      display: flex;
      align-items: center;
      gap: 16px;

      h1 {
        margin: 0;
        font-size: 28px;
        font-weight: bold;
        color: #303133;
      }
    }

    .property-actions {
      display: flex;
      gap: 10px;
    }
  }
}

.info-card {
  .card-title {
    font-size: 16px;
    font-weight: bold;
    color: #303133;
  }

  .remark-content {
    min-height: 120px;
    padding: 10px;
    color: #606266;
    line-height: 1.6;
  }
}

.log-card {
  margin-bottom: 0;

  h4 {
    margin: 0 0 8px 0;
    font-size: 14px;
    color: #303133;
  }

  p {
    margin: 4px 0;
    font-size: 13px;
    color: #606266;

    &.reason {
      color: #909399;
    }

    &.operator {
      color: #909399;
      font-size: 12px;
    }
  }
}

.empty-tip {
  text-align: center;
  padding: 40px 0;
  color: #909399;
}
</style>
