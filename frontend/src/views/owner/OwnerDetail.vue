<template>
  <div class="owner-detail">
    <div class="detail-header">
      <el-button @click="handleBack">
        <el-icon><ArrowLeft /></el-icon>返回
      </el-button>
    </div>

    <div v-loading="loading" class="detail-content">
      <div class="owner-header">
        <div class="owner-title">
          <el-avatar :size="64" :src="owner.avatar">
            {{ owner.ownerName?.charAt(0) || '业' }}
          </el-avatar>
          <div class="owner-info">
            <h1>{{ owner.ownerName }}</h1>
            <div class="owner-tags">
              <el-tag :type="owner.ownerType === 1 ? 'primary' : 'warning'" size="large">
                {{ owner.ownerType === 1 ? '个人业主' : '企业业主' }}
              </el-tag>
              <el-tag :type="getAuditTagType(owner.auditStatus)" size="large">
                {{ getAuditStatusLabel(owner.auditStatus) }}
              </el-tag>
            </div>
          </div>
        </div>
        <div class="owner-actions">
          <el-button v-if="hasEditPermission" type="primary" @click="handleEdit">
            <el-icon><Edit /></el-icon>编辑业主
          </el-button>
          <el-button v-if="hasAuditPermission && owner.auditStatus === 1" type="warning" @click="handleAudit">
            <el-icon><Promotion /></el-icon>审核认证
          </el-button>
          <el-button v-if="hasBindPropertyPermission" type="success" @click="handleBindProperty">
            <el-icon><Link /></el-icon>关联房产
          </el-button>
          <el-button v-if="hasDeletePermission" type="danger" @click="handleDelete">
            <el-icon><Delete /></el-icon>删除业主
          </el-button>
        </div>
      </div>

      <el-row :gutter="20">
        <el-col :span="12">
          <el-card class="info-card">
            <template #header>
              <span class="card-title">基本信息</span>
            </template>
            <el-descriptions :column="2" border>
              <el-descriptions-item label="业主姓名">{{ owner.ownerName || '-' }}</el-descriptions-item>
              <el-descriptions-item label="业主类型">
                {{ owner.ownerType === 1 ? '个人' : '企业' }}
              </el-descriptions-item>
              <el-descriptions-item label="联系电话">{{ owner.phone || '-' }}</el-descriptions-item>
              <el-descriptions-item label="电子邮箱">{{ owner.email || '-' }}</el-descriptions-item>
              <el-descriptions-item label="证件类型">{{ owner.idCardType || '身份证' }}</el-descriptions-item>
              <el-descriptions-item label="证件号码">{{ formatIdCard(owner.idCard) }}</el-descriptions-item>
              <el-descriptions-item label="性别">{{ owner.gender === 1 ? '男' : owner.gender === 2 ? '女' : '-' }}</el-descriptions-item>
              <el-descriptions-item label="出生日期">{{ owner.birthDate || '-' }}</el-descriptions-item>
              <el-descriptions-item label="联系地址" :span="2">{{ owner.address || '-' }}</el-descriptions-item>
            </el-descriptions>
          </el-card>
        </el-col>

        <el-col :span="12">
          <el-card class="info-card" v-if="owner.ownerType === 2">
            <template #header>
              <span class="card-title">企业信息</span>
            </template>
            <el-descriptions :column="2" border>
              <el-descriptions-item label="企业名称">{{ owner.enterpriseName || '-' }}</el-descriptions-item>
              <el-descriptions-item label="统一社会信用代码">{{ owner.creditCode || '-' }}</el-descriptions-item>
              <el-descriptions-item label="法定代表人">{{ owner.legalPerson || '-' }}</el-descriptions-item>
              <el-descriptions-item label="企业类型">{{ owner.enterpriseType || '-' }}</el-descriptions-item>
              <el-descriptions-item label="注册资本">{{ owner.registeredCapital || '-' }}</el-descriptions-item>
              <el-descriptions-item label="成立日期">{{ owner.establishDate || '-' }}</el-descriptions-item>
              <el-descriptions-item label="经营范围" :span="2">{{ owner.businessScope || '-' }}</el-descriptions-item>
              <el-descriptions-item label="注册地址" :span="2">{{ owner.registeredAddress || '-' }}</el-descriptions-item>
            </el-descriptions>
          </el-card>

          <el-card class="info-card" v-else>
            <template #header>
              <span class="card-title">紧急联系人</span>
            </template>
            <el-descriptions :column="2" border>
              <el-descriptions-item label="紧急联系人">{{ owner.emergencyContact || '-' }}</el-descriptions-item>
              <el-descriptions-item label="联系电话">{{ owner.emergencyPhone || '-' }}</el-descriptions-item>
              <el-descriptions-item label="关系">{{ owner.emergencyRelation || '-' }}</el-descriptions-item>
              <el-descriptions-item label="职业">{{ owner.occupation || '-' }}</el-descriptions-item>
              <el-descriptions-item label="工作单位">{{ owner.workUnit || '-' }}</el-descriptions-item>
              <el-descriptions-item label="学历">{{ owner.education || '-' }}</el-descriptions-item>
            </el-descriptions>
          </el-card>
        </el-col>
      </el-row>

      <el-row :gutter="20" style="margin-top: 20px;">
        <el-col :span="12">
          <el-card class="info-card">
            <template #header>
              <span class="card-title">认证信息</span>
            </template>
            <el-descriptions :column="2" border>
              <el-descriptions-item label="认证状态">
                <el-tag :type="getAuditTagType(owner.auditStatus)" size="small">
                  {{ getAuditStatusLabel(owner.auditStatus) }}
                </el-tag>
              </el-descriptions-item>
              <el-descriptions-item label="认证时间">{{ owner.auditTime || '-' }}</el-descriptions-item>
              <el-descriptions-item label="认证审核人">{{ owner.auditorName || '-' }}</el-descriptions-item>
              <el-descriptions-item label="注册时间">{{ owner.createTime || '-' }}</el-descriptions-item>
              <el-descriptions-item label="审核意见" :span="2">{{ owner.auditRemark || '-' }}</el-descriptions-item>
            </el-descriptions>
          </el-card>
        </el-col>

        <el-col :span="12">
          <el-card class="info-card">
            <template #header>
              <span class="card-title">备注信息</span>
            </template>
            <div class="remark-content">
              {{ owner.remark || '暂无备注' }}
            </div>
          </el-card>
        </el-col>
      </el-row>

      <el-card class="info-card" style="margin-top: 20px;">
        <template #header>
          <div class="card-header">
            <span class="card-title">关联房产</span>
            <el-button v-if="hasBindPropertyPermission" size="small" type="primary" @click="handleBindProperty">
              <el-icon><Plus /></el-icon>关联房产
            </el-button>
          </div>
        </template>
        <el-table :data="propertyList" border stripe>
          <el-table-column prop="propertyCode" label="房产编号" width="120" />
          <el-table-column prop="buildingName" label="楼宇" width="120" />
          <el-table-column prop="floorNumber" label="楼层" width="80" />
          <el-table-column prop="buildingArea" label="建筑面积(㎡)" width="120" />
          <el-table-column prop="houseType" label="户型" width="100" />
          <el-table-column label="产权比例" width="100">
            <template #default="{ row }">
              {{ row.ownerShipRatio || '100%' }}
            </template>
          </el-table-column>
          <el-table-column label="共有方式" width="100">
            <template #default="{ row }">
              {{ row.ownershipType === 1 ? '单独所有' : row.ownershipType === 2 ? '共同共有' : '按份共有' }}
            </template>
          </el-table-column>
          <el-table-column prop="bindTime" label="关联时间" width="160" />
          <el-table-column label="操作" width="120" fixed="right">
            <template #default="{ row }">
              <el-button v-if="hasUnbindPropertyPermission" type="danger" size="small" @click="handleUnbindProperty(row)">
                解除关联
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <div v-if="propertyList.length === 0" class="empty-tip">
          暂无关联房产
        </div>
      </el-card>

      <el-card class="info-card" style="margin-top: 20px;">
        <template #header>
          <span class="card-title">审核记录</span>
        </template>
        <el-timeline>
          <el-timeline-item
            v-for="(log, index) in auditLogs"
            :key="log.id"
            :timestamp="log.createTime"
            :type="index === 0 ? 'primary' : ''"
            placement="top">
            <el-card shadow="never" class="log-card">
              <h4>
                <el-tag :type="getAuditTagType(log.auditStatus)" size="small">
                  {{ getAuditStatusLabel(log.auditStatus) }}
                </el-tag>
              </h4>
              <p v-if="log.auditRemark" class="reason">审核意见：{{ log.auditRemark }}</p>
              <p class="operator">操作人：{{ log.auditorName || '系统' }}</p>
            </el-card>
          </el-timeline-item>
        </el-timeline>
        <div v-if="auditLogs.length === 0" class="empty-tip">
          暂无审核记录
        </div>
      </el-card>
    </div>

    <el-dialog v-model="auditDialogVisible" title="审核认证" width="500px">
      <el-form label-width="100px">
        <el-form-item label="业主姓名">
          <span>{{ owner?.ownerName }}</span>
        </el-form-item>
        <el-form-item label="当前状态">
          <el-tag :type="getAuditTagType(owner?.auditStatus)">
            {{ getAuditStatusLabel(owner?.auditStatus) }}
          </el-tag>
        </el-form-item>
        <el-form-item label="审核结果">
          <el-radio-group v-model="auditResult">
            <el-radio :value="2">通过</el-radio>
            <el-radio :value="3">拒绝</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="审核意见">
          <el-input v-model="auditRemark" type="textarea" :rows="3" placeholder="请输入审核意见（拒绝时必填）" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="auditDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="confirmAudit">确定</el-button>
      </template>
    </el-dialog>

    <el-dialog v-model="bindDialogVisible" title="关联房产" width="600px" @close="handleBindClose">
      <el-form label-width="100px">
        <el-form-item label="选择房产">
          <el-select v-model="selectedPropertyId" placeholder="请选择要关联的房产" style="width: 100%;" filterable>
            <el-option 
              v-for="p in unboundPropertyList" 
              :key="p.id" 
              :label="`${p.propertyCode} - ${p.buildingName} ${p.floorNumber}层`" 
              :value="p.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="产权比例">
          <el-input-number v-model="ownershipRatio" :min="1" :max="100" style="width: 100px;" />
          <span style="margin-left: 10px;">%</span>
        </el-form-item>
        <el-form-item label="共有方式">
          <el-radio-group v-model="ownershipType">
            <el-radio :value="1">单独所有</el-radio>
            <el-radio :value="2">共同共有</el-radio>
            <el-radio :value="3">按份共有</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注">
          <el-input v-model="bindRemark" type="textarea" :rows="2" placeholder="请输入备注（可选）" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="bindDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="confirmBind">确定关联</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { ElMessage, ElMessageBox } from 'element-plus'
import { ArrowLeft, Edit, Delete, Promotion, Link, Plus } from '@element-plus/icons-vue'
import api from '@/api'

const route = useRoute()
const router = useRouter()
const userStore = useUserStore()

const loading = ref(false)
const owner = ref({})
const propertyList = ref([])
const auditLogs = ref([])

const auditDialogVisible = ref(false)
const auditResult = ref(2)
const auditRemark = ref('')

const bindDialogVisible = ref(false)
const unboundPropertyList = ref([])
const selectedPropertyId = ref(null)
const ownershipRatio = ref(100)
const ownershipType = ref(1)
const bindRemark = ref('')

const hasEditPermission = computed(() => userStore.hasPermission('park:owner:edit'))
const hasDeletePermission = computed(() => userStore.hasPermission('park:owner:delete'))
const hasAuditPermission = computed(() => userStore.hasPermission('park:owner:audit'))
const hasBindPropertyPermission = computed(() => userStore.hasPermission('park:owner:bindProperty'))
const hasUnbindPropertyPermission = computed(() => userStore.hasPermission('park:owner:unbindProperty'))

onMounted(() => {
  loadDetail()
})

function loadDetail() {
  const id = route.params.id
  if (!id) return

  loading.value = true
  api.parkOwner.getById(id).then(res => {
    owner.value = res.data || {}
    propertyList.value = res.data.propertyList || []
    auditLogs.value = res.data.auditLogs || []
  }).finally(() => {
    loading.value = false
  })
}

function formatIdCard(idCard) {
  if (!idCard) return '-'
  if (idCard.length === 18) {
    return idCard.substring(0, 6) + '********' + idCard.substring(14)
  }
  return idCard
}

function getAuditStatusLabel(status) {
  const map = {
    0: '未认证',
    1: '待审核',
    2: '已通过',
    3: '已拒绝'
  }
  return map[status] || '-'
}

function getAuditTagType(status) {
  const map = {
    0: 'info',
    1: 'warning',
    2: 'success',
    3: 'danger'
  }
  return map[status] || 'info'
}

function handleBack() {
  router.back()
}

function handleEdit() {
  router.push(`/park/owner/edit/${owner.value.id}`)
}

function handleDelete() {
  ElMessageBox.confirm(`确定要删除业主 "${owner.value.ownerName}" 吗？`, '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    api.parkOwner.delete(owner.value.id).then(() => {
      ElMessage.success('删除成功')
      router.back()
    })
  }).catch(() => {})
}

function handleAudit() {
  auditResult.value = 2
  auditRemark.value = ''
  auditDialogVisible.value = true
}

function confirmAudit() {
  if (auditResult.value === 3 && !auditRemark.value.trim()) {
    ElMessage.warning('拒绝时请填写审核意见')
    return
  }
  api.parkOwner.audit({
    id: owner.value.id,
    auditStatus: auditResult.value,
    auditRemark: auditRemark.value
  }).then(() => {
    ElMessage.success('审核完成')
    auditDialogVisible.value = false
    loadDetail()
  })
}

function handleBindProperty() {
  selectedPropertyId.value = null
  ownershipRatio.value = 100
  ownershipType.value = 1
  bindRemark.value = ''
  api.parkOwner.getUnboundProperties().then(res => {
    unboundPropertyList.value = res.data.list || []
    bindDialogVisible.value = true
  })
}

function handleBindClose() {
  unboundPropertyList.value = []
}

function confirmBind() {
  if (!selectedPropertyId.value) {
    ElMessage.warning('请选择要关联的房产')
    return
  }
  api.parkOwner.bindProperty({
    ownerId: owner.value.id,
    propertyId: selectedPropertyId.value,
    ownershipRatio: ownershipRatio.value,
    ownershipType: ownershipType.value,
    remark: bindRemark.value
  }).then(() => {
    ElMessage.success('关联成功')
    bindDialogVisible.value = false
    loadDetail()
  })
}

function handleUnbindProperty(row) {
  ElMessageBox.confirm(`确定要解除与房产 "${row.propertyCode}" 的关联吗？`, '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    api.parkOwner.unbindProperty({
      ownerId: owner.value.id,
      propertyId: row.id
    }).then(() => {
      ElMessage.success('解除关联成功')
      loadDetail()
    })
  }).catch(() => {})
}
</script>

<style scoped lang="scss">
.owner-detail {
  padding: 20px;
}

.detail-header {
  margin-bottom: 20px;
}

.detail-content {
  .owner-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    padding: 20px;
    background: #fff;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);

    .owner-title {
      display: flex;
      align-items: center;
      gap: 16px;

      .owner-info {
        h1 {
          margin: 0 0 8px 0;
          font-size: 28px;
          font-weight: bold;
          color: #303133;
        }

        .owner-tags {
          display: flex;
          gap: 8px;
        }
      }
    }

    .owner-actions {
      display: flex;
      gap: 10px;
    }
  }
}

.info-card {
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

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
