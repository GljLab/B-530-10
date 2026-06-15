<template>
  <div class="owner-form">
    <div class="form-header">
      <el-button @click="handleBack">
        <el-icon><ArrowLeft /></el-icon>返回
      </el-button>
    </div>

    <el-card class="form-card" v-loading="loading">
      <template #header>
        <div class="card-header">
          <span>{{ pageTitle }}</span>
        </div>
      </template>

      <el-steps :active="activeStep" finish-status="success" align-center>
        <el-step title="选择业主类型" />
        <el-step :title="formData.ownerType === 1 ? '个人信息' : '企业信息'" />
        <el-step title="联系信息" />
        <el-step title="关联房产" />
      </el-steps>

      <el-form ref="formRef" :model="formData" :rules="formRules" label-width="120px" class="form-content">
        <div v-if="activeStep === 0" class="step-content">
          <div class="type-selector">
            <div 
              class="type-card" 
              :class="{ active: formData.ownerType === 1 }"
              @click="selectOwnerType(1)">
              <el-icon :size="48" class="type-icon"><User /></el-icon>
              <h3>个人业主</h3>
              <p>适用于个人、家庭业主</p>
            </div>
            <div 
              class="type-card" 
              :class="{ active: formData.ownerType === 2 }"
              @click="selectOwnerType(2)">
              <el-icon :size="48" class="type-icon"><OfficeBuilding /></el-icon>
              <h3>企业业主</h3>
              <p>适用于公司、企业单位</p>
            </div>
          </div>
        </div>

        <div v-if="activeStep === 1" class="step-content">
          <div v-if="formData.ownerType === 1">
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="业主姓名" prop="ownerName">
                  <el-input v-model="formData.ownerName" placeholder="请输入业主姓名" />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="性别" prop="gender">
                  <el-radio-group v-model="formData.gender">
                    <el-radio :value="1">男</el-radio>
                    <el-radio :value="2">女</el-radio>
                  </el-radio-group>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="证件类型" prop="idCardType">
                  <el-select v-model="formData.idCardType" placeholder="请选择证件类型" style="width: 100%;">
                    <el-option label="身份证" value="身份证" />
                    <el-option label="护照" value="护照" />
                    <el-option label="港澳通行证" value="港澳通行证" />
                    <el-option label="台胞证" value="台胞证" />
                    <el-option label="其他" value="其他" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="证件号码" prop="idCard">
                  <el-input v-model="formData.idCard" placeholder="请输入证件号码" />
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="出生日期" prop="birthDate">
                  <el-date-picker 
                    v-model="formData.birthDate" 
                    type="date" 
                    placeholder="选择出生日期" 
                    style="width: 100%;" />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="学历" prop="education">
                  <el-select v-model="formData.education" placeholder="请选择学历" clearable style="width: 100%;">
                    <el-option label="小学" value="小学" />
                    <el-option label="初中" value="初中" />
                    <el-option label="高中" value="高中" />
                    <el-option label="大专" value="大专" />
                    <el-option label="本科" value="本科" />
                    <el-option label="硕士" value="硕士" />
                    <el-option label="博士" value="博士" />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="职业" prop="occupation">
                  <el-input v-model="formData.occupation" placeholder="请输入职业" clearable />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="工作单位" prop="workUnit">
                  <el-input v-model="formData.workUnit" placeholder="请输入工作单位" clearable />
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="紧急联系人" prop="emergencyContact">
                  <el-input v-model="formData.emergencyContact" placeholder="请输入紧急联系人" clearable />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="紧急联系电话" prop="emergencyPhone">
                  <el-input v-model="formData.emergencyPhone" placeholder="请输入紧急联系电话" clearable />
                </el-form-item>
              </el-col>
            </el-row>
            <el-form-item label="与紧急联系人关系" prop="emergencyRelation">
              <el-select v-model="formData.emergencyRelation" placeholder="请选择关系" clearable style="width: 100%;">
                <el-option label="配偶" value="配偶" />
                <el-option label="父母" value="父母" />
                <el-option label="子女" value="子女" />
                <el-option label="兄弟姐妹" value="兄弟姐妹" />
                <el-option label="朋友" value="朋友" />
                <el-option label="同事" value="同事" />
                <el-option label="其他" value="其他" />
              </el-select>
            </el-form-item>
          </div>

          <div v-else>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="企业名称" prop="enterpriseName">
                  <el-input v-model="formData.enterpriseName" placeholder="请输入企业名称" />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="统一社会信用代码" prop="creditCode">
                  <el-input v-model="formData.creditCode" placeholder="请输入统一社会信用代码" />
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="法定代表人" prop="legalPerson">
                  <el-input v-model="formData.legalPerson" placeholder="请输入法定代表人姓名" />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="法人证件号" prop="legalPersonIdCard">
                  <el-input v-model="formData.legalPersonIdCard" placeholder="请输入法人证件号" clearable />
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="企业类型" prop="enterpriseType">
                  <el-select v-model="formData.enterpriseType" placeholder="请选择企业类型" clearable style="width: 100%;">
                    <el-option label="国有企业" value="国有企业" />
                    <el-option label="集体企业" value="集体企业" />
                    <el-option label="私营企业" value="私营企业" />
                    <el-option label="联营企业" value="联营企业" />
                    <el-option label="股份制企业" value="股份制企业" />
                    <el-option label="外商投资企业" value="外商投资企业" />
                    <el-option label="其他" value="其他" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="注册资本" prop="registeredCapital">
                  <el-input v-model="formData.registeredCapital" placeholder="请输入注册资本" clearable>
                    <template #append>万元</template>
                  </el-input>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="12">
                <el-form-item label="成立日期" prop="establishDate">
                  <el-date-picker 
                    v-model="formData.establishDate" 
                    type="date" 
                    placeholder="选择成立日期" 
                    style="width: 100%;" />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="联系人" prop="contactPerson">
                  <el-input v-model="formData.contactPerson" placeholder="请输入联系人姓名" />
                </el-form-item>
              </el-col>
            </el-row>
            <el-form-item label="经营范围" prop="businessScope">
              <el-input v-model="formData.businessScope" type="textarea" :rows="3" placeholder="请输入经营范围" clearable />
            </el-form-item>
            <el-form-item label="注册地址" prop="registeredAddress">
              <el-input v-model="formData.registeredAddress" type="textarea" :rows="2" placeholder="请输入注册地址" clearable />
            </el-form-item>
          </div>
        </div>

        <div v-if="activeStep === 2" class="step-content">
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item label="联系电话" prop="phone">
                <el-input v-model="formData.phone" placeholder="请输入联系电话" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="电子邮箱" prop="email">
                <el-input v-model="formData.email" placeholder="请输入电子邮箱" clearable />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item label="微信号" prop="wechat">
                <el-input v-model="formData.wechat" placeholder="请输入微信号" clearable />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="QQ号" prop="qq">
                <el-input v-model="formData.qq" placeholder="请输入QQ号" clearable />
              </el-form-item>
            </el-col>
          </el-row>
          <el-form-item label="联系地址" prop="address">
            <el-input v-model="formData.address" type="textarea" :rows="2" placeholder="请输入联系地址" />
          </el-form-item>
          <el-form-item label="备注" prop="remark">
            <el-input v-model="formData.remark" type="textarea" :rows="3" placeholder="请输入备注信息" clearable />
          </el-form-item>
        </div>

        <div v-if="activeStep === 3" class="step-content">
          <div class="property-select-header">
            <span>选择关联房产（可选）</span>
            <el-button type="primary" size="small" @click="handleAddProperty">
              <el-icon><Plus /></el-icon>添加房产
            </el-button>
          </div>
          <el-table :data="selectedProperties" border stripe style="margin-top: 15px;">
            <el-table-column prop="propertyCode" label="房产编号" width="120" />
            <el-table-column prop="buildingName" label="楼宇" width="120" />
            <el-table-column prop="floorNumber" label="楼层" width="80" />
            <el-table-column prop="buildingArea" label="建筑面积(㎡)" width="120" />
            <el-table-column label="产权比例" width="150">
              <template #default="{ row }">
                <el-input-number v-model="row.ownershipRatio" :min="1" :max="100" size="small" />
                <span style="margin-left: 5px;">%</span>
              </template>
            </el-table-column>
            <el-table-column label="共有方式" width="180">
              <template #default="{ row }">
                <el-select v-model="row.ownershipType" size="small" style="width: 100%;">
                  <el-option label="单独所有" :value="1" />
                  <el-option label="共同共有" :value="2" />
                  <el-option label="按份共有" :value="3" />
                </el-select>
              </template>
            </el-table-column>
            <el-table-column label="操作" width="100">
              <template #default="{ $index }">
                <el-button type="danger" size="small" @click="handleRemoveProperty($index)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
          <div v-if="selectedProperties.length === 0" class="empty-tip">
            暂无关联房产，可点击"添加房产"按钮进行关联
          </div>
        </div>
      </el-form>

      <div class="form-actions">
        <el-button @click="handleBack">取消</el-button>
        <el-button v-if="activeStep > 0" @click="prevStep">上一步</el-button>
        <el-button v-if="activeStep < 3" type="primary" @click="nextStep">下一步</el-button>
        <el-button v-if="activeStep === 3" type="primary" @click="handleSubmit">提交</el-button>
      </div>
    </el-card>

    <el-dialog v-model="propertyDialogVisible" title="选择房产" width="800px">
      <el-form :inline="true" :model="propertyQuery" size="default">
        <el-form-item label="房产编号">
          <el-input v-model="propertyQuery.propertyCode" placeholder="请输入" clearable @keyup.enter="searchProperty" />
        </el-form-item>
        <el-form-item label="楼宇">
          <el-select v-model="propertyQuery.buildingId" placeholder="全部" clearable style="width: 140px;">
            <el-option v-for="b in buildingList" :key="b.id" :label="b.buildingName" :value="b.id" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchProperty">
            <el-icon><Search /></el-icon>搜索
          </el-button>
        </el-form-item>
      </el-form>
      <el-table 
        :data="propertyList" 
        border 
        stripe 
        height="300"
        @selection-change="handlePropertySelectionChange">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="propertyCode" label="房产编号" width="120" />
        <el-table-column prop="buildingName" label="楼宇" width="120" />
        <el-table-column prop="floorNumber" label="楼层" width="80" />
        <el-table-column prop="buildingArea" label="建筑面积(㎡)" width="120" />
        <el-table-column prop="houseType" label="户型" width="100" />
        <el-table-column label="状态" width="100">
          <template #default="{ row }">
            <el-tag :type="row.status === 3 ? 'success' : 'warning'" size="small">
              {{ row.status === 3 ? '可关联' : '待确认' }}
            </el-tag>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        class="property-pagination"
        v-model:current-page="propertyQuery.pageNum"
        v-model:page-size="propertyQuery.pageSize"
        :page-sizes="[5, 10, 20]"
        :total="propertyTotal"
        layout="total, prev, pager, next"
        @current-change="searchProperty" />
      <template #footer>
        <el-button @click="propertyDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="confirmAddProperty">确定添加</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import { ArrowLeft, User, OfficeBuilding, Plus, Search } from '@element-plus/icons-vue'
import api from '@/api'

const route = useRoute()
const router = useRouter()

const loading = ref(false)
const isEdit = ref(false)
const activeStep = ref(0)
const formRef = ref(null)

const pageTitle = computed(() => isEdit.value ? '编辑业主' : '新增业主')

const formData = reactive({
  id: null,
  ownerType: 1,
  ownerName: '',
  gender: 1,
  idCardType: '身份证',
  idCard: '',
  birthDate: null,
  education: '',
  occupation: '',
  workUnit: '',
  emergencyContact: '',
  emergencyPhone: '',
  emergencyRelation: '',
  enterpriseName: '',
  creditCode: '',
  legalPerson: '',
  legalPersonIdCard: '',
  enterpriseType: '',
  registeredCapital: '',
  establishDate: null,
  contactPerson: '',
  businessScope: '',
  registeredAddress: '',
  phone: '',
  email: '',
  wechat: '',
  qq: '',
  address: '',
  remark: ''
})

const personalRules = {
  ownerName: [{ required: true, message: '请输入业主姓名', trigger: 'blur' }],
  gender: [{ required: true, message: '请选择性别', trigger: 'change' }],
  idCardType: [{ required: true, message: '请选择证件类型', trigger: 'change' }],
  idCard: [{ required: true, message: '请输入证件号码', trigger: 'blur' }],
  birthDate: [{ required: true, message: '请选择出生日期', trigger: 'change' }]
}

const enterpriseRules = {
  enterpriseName: [{ required: true, message: '请输入企业名称', trigger: 'blur' }],
  creditCode: [{ required: true, message: '请输入统一社会信用代码', trigger: 'blur' }],
  legalPerson: [{ required: true, message: '请输入法定代表人', trigger: 'blur' }],
  contactPerson: [{ required: true, message: '请输入联系人', trigger: 'blur' }]
}

const commonRules = {
  phone: [
    { required: true, message: '请输入联系电话', trigger: 'blur' },
    { pattern: /^1[3-9]\d{9}$/, message: '请输入正确的手机号码', trigger: 'blur' }
  ],
  email: [
    { type: 'email', message: '请输入正确的邮箱地址', trigger: 'blur' }
  ],
  address: [{ required: true, message: '请输入联系地址', trigger: 'blur' }]
}

const formRules = computed(() => {
  if (formData.ownerType === 1) {
    return { ...personalRules, ...commonRules }
  } else {
    return { ...enterpriseRules, ...commonRules }
  }
})

const selectedProperties = ref([])
const propertyDialogVisible = ref(false)
const propertyList = ref([])
const propertyTotal = ref(0)
const propertySelectedIds = ref([])
const buildingList = ref([])

const propertyQuery = reactive({
  pageNum: 1,
  pageSize: 10,
  propertyCode: '',
  buildingId: null
})

onMounted(() => {
  loadBuildingList()
  if (route.params.id) {
    isEdit.value = true
    loadDetail()
  }
})

function loadBuildingList() {
  api.parkBuilding.listAll({ parkId: 1 }).then(res => {
    buildingList.value = res.data || []
  })
}

function loadDetail() {
  loading.value = true
  api.parkOwner.getById(route.params.id).then(res => {
    const data = res.data || {}
    Object.assign(formData, data)
    selectedProperties.value = data.propertyList?.map(p => ({
      id: p.id,
      propertyCode: p.propertyCode,
      buildingName: p.buildingName,
      floorNumber: p.floorNumber,
      buildingArea: p.buildingArea,
      ownershipRatio: p.ownershipRatio || 100,
      ownershipType: p.ownershipType || 1
    })) || []
  }).finally(() => {
    loading.value = false
  })
}

function selectOwnerType(type) {
  formData.ownerType = type
}

function prevStep() {
  if (activeStep.value > 0) {
    activeStep.value--
  }
}

async function nextStep() {
  const fields = getStepFields()
  if (fields && fields.length > 0) {
    try {
      await formRef.value.validateField(fields)
    } catch {
      return
    }
  }
  if (activeStep.value < 3) {
    activeStep.value++
  }
}

function getStepFields() {
  if (activeStep.value === 0) {
    return null
  } else if (activeStep.value === 1) {
    if (formData.ownerType === 1) {
      return ['ownerName', 'gender', 'idCardType', 'idCard', 'birthDate']
    } else {
      return ['enterpriseName', 'creditCode', 'legalPerson', 'contactPerson']
    }
  } else if (activeStep.value === 2) {
    return ['phone', 'address']
  }
  return null
}

function handleAddProperty() {
  propertyQuery.pageNum = 1
  propertyQuery.propertyCode = ''
  propertyQuery.buildingId = null
  propertySelectedIds.value = []
  searchProperty()
  propertyDialogVisible.value = true
}

function searchProperty() {
  const params = {
    pageNum: propertyQuery.pageNum,
    pageSize: propertyQuery.pageSize,
    propertyCode: propertyQuery.propertyCode || undefined,
    buildingId: propertyQuery.buildingId || undefined,
    unbound: true
  }
  api.parkOwner.getUnboundProperties(params).then(res => {
    propertyList.value = res.data.list || []
    propertyTotal.value = res.data.total || 0
  })
}

function handlePropertySelectionChange(selection) {
  propertySelectedIds.value = selection.map(item => item.id)
}

function confirmAddProperty() {
  if (propertySelectedIds.value.length === 0) {
    ElMessage.warning('请选择要关联的房产')
    return
  }
  const newProperties = propertyList.value
    .filter(p => propertySelectedIds.value.includes(p.id))
    .filter(p => !selectedProperties.value.find(sp => sp.id === p.id))
    .map(p => ({
      id: p.id,
      propertyCode: p.propertyCode,
      buildingName: p.buildingName,
      floorNumber: p.floorNumber,
      buildingArea: p.buildingArea,
      ownershipRatio: 100,
      ownershipType: 1
    }))
  selectedProperties.value.push(...newProperties)
  propertyDialogVisible.value = false
  ElMessage.success(`已添加 ${newProperties.length} 套房产`)
}

function handleRemoveProperty(index) {
  selectedProperties.value.splice(index, 1)
}

async function handleSubmit() {
  try {
    await formRef.value.validate()
  } catch {
    return
  }

  const submitData = { 
    ...formData,
    propertyList: selectedProperties.value.map(p => ({
      propertyId: p.id,
      ownershipRatio: p.ownershipRatio,
      ownershipType: p.ownershipType
    }))
  }

  if (isEdit.value) {
    api.parkOwner.update(submitData).then(() => {
      ElMessage.success('修改成功')
      router.back()
    })
  } else {
    api.parkOwner.add(submitData).then(() => {
      ElMessage.success('新增成功')
      router.back()
    })
  }
}

function handleBack() {
  ElMessageBox.confirm('确定要离开吗？未保存的数据将会丢失。', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    router.back()
  }).catch(() => {})
}
</script>

<style scoped lang="scss">
.owner-form {
  padding: 20px;
}

.form-header {
  margin-bottom: 20px;
}

.form-card {
  .card-header {
    font-size: 18px;
    font-weight: bold;
    color: #303133;
  }

  .form-content {
    padding: 30px 20px;
  }

  .step-content {
    min-height: 400px;
    padding: 20px 0;
  }

  .form-actions {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    padding-top: 20px;
    border-top: 1px solid #ebeef5;
  }
}

.type-selector {
  display: flex;
  justify-content: center;
  gap: 40px;
  padding: 40px 0;

  .type-card {
    width: 280px;
    padding: 40px;
    border: 2px solid #ebeef5;
    border-radius: 8px;
    text-align: center;
    cursor: pointer;
    transition: all 0.3s;

    &:hover {
      border-color: #c6e2ff;
      background: #f0f7ff;
    }

    &.active {
      border-color: #409EFF;
      background: #ecf5ff;
    }

    .type-icon {
      color: #909399;
      margin-bottom: 15px;
    }

    &.active .type-icon {
      color: #409EFF;
    }

    h3 {
      margin: 0 0 10px 0;
      font-size: 18px;
      color: #303133;
    }

    p {
      margin: 0;
      font-size: 14px;
      color: #909399;
    }
  }
}

.property-select-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 16px;
  font-weight: bold;
  color: #303133;
}

.property-pagination {
  margin-top: 15px;
  text-align: right;
}

.empty-tip {
  text-align: center;
  padding: 40px 0;
  color: #909399;
}
</style>
