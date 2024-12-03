<template>
  <div style="margin-bottom: 20px">
    <el-button>默认</el-button>
    <el-button type="primary">主要</el-button>
    <el-button type="success">成功</el-button>
    <el-button type="info">信息</el-button>
    <el-button type="warning">警告</el-button>
    <el-button type="danger">危险</el-button>
  </div>
  <div>
    <el-button :icon="Search" circle />
    <el-button type="primary" :icon="Edit" circle />
    <el-button type="success" :icon="Check" circle />
    <el-button type="info" :icon="Message" circle />
    <el-button type="warning" :icon="Star" circle />
    <el-button type="danger" :icon="Delete" circle />
  </div>
  <div class="container">
    <el-form ref="formRef" :model="form" :rules="rules" label-width="120px" label-position="right" status-icon>
      <!-- 基本信息 -->
      <el-divider>基本信息</el-divider>
      <el-row :gutter="20">
        <el-col :span="12">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="form.username" placeholder="请输入用户名" />
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="密码" prop="password">
            <el-input v-model="form.password" type="password" show-password placeholder="请输入密码" />
          </el-form-item>
        </el-col>
      </el-row>

      <!-- 个人信息 -->
      <el-divider>个人信息</el-divider>
      <el-row :gutter="20">
        <el-col :span="12">
          <el-form-item label="性别" prop="gender">
            <el-radio-group v-model="form.gender">
              <el-radio label="male">男</el-radio>
              <el-radio label="female">女</el-radio>
              <el-radio label="other">其他</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="出生日期" prop="birthday">
            <el-date-picker v-model="form.birthday" type="date" placeholder="选择出生日期" style="width: 100%" />
          </el-form-item>
        </el-col>
      </el-row>

      <el-form-item label="兴趣爱好" prop="hobbies">
        <el-checkbox-group v-model="form.hobbies">
          <el-checkbox label="reading">阅读</el-checkbox>
          <el-checkbox label="sports">运动</el-checkbox>
          <el-checkbox label="music">音乐</el-checkbox>
          <el-checkbox label="travel">旅行</el-checkbox>
          <el-checkbox label="cooking">烹饪</el-checkbox>
        </el-checkbox-group>
      </el-form-item>

      <!-- 联系方式 -->
      <el-divider>联系方式</el-divider>

      <el-form-item label="详细地址" prop="address">
        <el-input v-model="form.address" type="textarea" :rows="2" placeholder="请输入详细地址" />
      </el-form-item>

      <!-- 其他设置 -->
      <el-divider>其他设置</el-divider>
      <el-form-item label="技能水平" prop="skillLevel">
        <el-slider v-model="form.skillLevel" :marks="{
          0: '初学者',
          25: '入门',
          50: '进阶',
          75: '熟练',
          100: '专家'
        }" />
      </el-form-item>


      <el-form-item label="头像" prop="avatar">
        <el-upload class="avatar-uploader" action="#" :show-file-list="false" :auto-upload="false"
          :on-change="handleAvatarChange">
          <img v-if="imageUrl" :src="imageUrl" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon">
            <Plus />
          </el-icon>
        </el-upload>
      </el-form-item>

      <el-form-item label="个人简介" prop="bio">
        <el-input v-model="form.bio" type="textarea" :rows="4" placeholder="请输入个人简介" />
      </el-form-item>

      <!-- 提交按钮 -->
      <el-form-item>
        <el-button type="primary" @click="submitForm(formRef)">提交</el-button>
        <el-button @click="resetForm(formRef)">重置</el-button>
      </el-form-item>
    </el-form>
  </div>

  <!-- 图表展示区域 -->
  <el-card class="chart-card">
    <div ref="chartRef" style="width: 100%; height: 400px;"></div>
  </el-card>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { Plus } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'
import * as echarts from 'echarts'

import {
  Check,
  Delete,
  Edit,
  Message,
  Search,
  Star,
} from '@element-plus/icons-vue'

const formRef = ref(null)
const imageUrl = ref('')

// 表单数据
const form = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
  gender: '',
  birthday: '',
  hobbies: [],
  region: [],
  address: '',
  skillLevel: 0,
  notifications: false,
  avatar: '',
  bio: ''
})

// 表单验证规则
const rules = reactive({
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱地址', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱地址', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, message: '密码长度不能小于6位', trigger: 'blur' }
  ],
  confirmPassword: [
    { required: true, message: '请确认密码', trigger: 'blur' },
    {
      validator: (rule, value, callback) => {
        if (value !== form.password) {
          callback(new Error('两次输入密码不一致'))
        } else {
          callback()
        }
      },
      trigger: 'blur'
    }
  ],
  gender: [
    { required: true, message: '请选择性别', trigger: 'change' }
  ],
  birthday: [
    { required: true, message: '请选择出生日期', trigger: 'change' }
  ],
  hobbies: [
    { type: 'array', required: true, message: '请至少选择一个兴趣爱好', trigger: 'change' }
  ],
  bio: [
    { required: true, message: '请输入个人简介', trigger: 'blur' }
  ]
})


// 处理头像上传
const handleAvatarChange = (file) => {
  imageUrl.value = URL.createObjectURL(file.raw)
  form.avatar = file.raw
}

// 提交表单
const submitForm = async (formEl) => {
  if (!formEl) return
  await formEl.validate((valid, fields) => {
    if (valid) {
      ElMessage.success('提交成功！')
      console.log('表单数据：', form)
    } else {
      console.log('验证失败', fields)
    }
  })
}

// 重置表单
const resetForm = (formEl) => {
  if (!formEl) return
  formEl.resetFields()
  imageUrl.value = ''
}

// 图表相关
const chartRef = ref(null)
let myChart = null

// 数据
const chartData = {
  dates: ['1月', '2月', '3月', '4月', '5月', '6月', '7月'],
  values: [
    {
      name: '访问量',
      data: [820, 932, 901, 934, 1290, 1330, 1320]
    },
    {
      name: '注册量',
      data: [420, 532, 501, 534, 690, 730, 820]
    }
  ]
}

// 初始化图表
const initChart = () => {
  if (chartRef.value) {
    myChart = echarts.init(chartRef.value)
    const option = {
      title: {
        text: '折线图'
      },
      tooltip: {
        trigger: 'axis'
      },
      legend: {
        data: ['访问量', '注册量']
      },
      grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
      },
      xAxis: {
        type: 'category',
        boundaryGap: false,
        data: chartData.dates
      },
      yAxis: {
        type: 'value'
      },
      series: chartData.values.map(item => ({
        name: item.name,
        type: 'line',
        data: item.data,
        smooth: true,
        areaStyle: {
          opacity: 0.1
        },
        emphasis: {
          focus: 'series'
        }
      }))
    }
    myChart.setOption(option)
  }
}

// 监听窗口大小变化
const handleResize = () => {
  if (myChart) {
    myChart.resize()
  }
}

// 图表类型
const chartType = ref('line')

// 处理图表类型切换
const handleChartTypeChange = (type) => {
  if (myChart) {
    const option = myChart.getOption()
    option.series = option.series.map(item => ({
      ...item,
      type: type
    }))
    myChart.setOption(option)
  }
}

onMounted(() => {
  initChart()
  window.addEventListener('resize', handleResize)
})
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 20px auto;
  padding: 0 20px;
}

.form-card {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.avatar-uploader {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  width: 178px;
  height: 178px;
}

.avatar-uploader:hover {
  border-color: #409eff;
}

.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
  line-height: 178px;
}

.avatar {
  width: 178px;
  height: 178px;
  display: block;
}

:deep(.el-upload) {
  width: 178px;
  height: 178px;
}

.chart-card {
  margin-top: 20px;
}

.chart-card .card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chart-card h2 {
  margin: 0;
  font-size: 18px;
}
</style>
