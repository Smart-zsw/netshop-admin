<template>
  <div class="tools-div">
    <el-button type="success" size="small" @click="addShow">添 加</el-button>
  </div>

  <el-dialog v-model="dialogVisible" title="添加或修改" width="30%">
    <el-form label-width="120px">
      <el-form-item label="品牌名称">
        <el-input v-model="brand.name" />
      </el-form-item>
      <el-form-item label="品牌图标">
        <el-upload
          class="avatar-uploader"
          action="http://localhost:8501/admin/system/fileUpload"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :headers="headers"
        >
          <img
            v-if="brand.logo"
            :src="brand.logo"
            class="avatar"
            style="max-width: 100px; max-height: 100px;"
          />
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
        </el-upload>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="saveOrUpdate">提交</el-button>
        <el-button @click="dialogVisible = false">取消</el-button>
      </el-form-item>
    </el-form>
  </el-dialog>

  <el-table :data="list" style="width: 100%">
    <el-table-column prop="name" label="品牌名称" />
    <el-table-column prop="logo" label="品牌图标" #default="scope">
      <img :src="scope.row.logo" width="50" />
    </el-table-column>
    <el-table-column prop="createTime" label="创建时间" />
    <el-table-column label="操作" align="center" width="200" #default="scope">
      <el-button type="primary" size="small" @click="editBrand(scope.row)">
        修改
      </el-button>
      <el-button type="danger" size="small" @click="deleteBrand(scope.row.id)">
        删除
      </el-button>
    </el-table-column>
  </el-table>

  <el-pagination
    v-model:current-page="pageParams.page"
    v-model:page-size="pageParams.limit"
    :page-sizes="[10, 20, 50, 100]"
    layout="total, sizes, prev, pager, next"
    :total="total"
    @size-change="handleSizeChange"
    @current-change="handleCurrentChange"
  />
</template>

<script setup>
import { ref, onMounted } from 'vue'
import {
  DeleteBrandById,
  GetBrandPageList,
  SaveBrand,
  UpdateBrandById,
} from '@/api/brand.js'
import { ElMessage, ElMessageBox } from 'element-plus'
import { useApp } from '@/pinia/modules/app'

const editBrand = brandData => {
  brand.value = { ...brandData } // 使用品牌数据填充表单
  dialogVisible.value = true // 显示对话框
}

//添加
const headers = {
  // 从pinia中获取token，在进行文件上传的时候将token设置到请求头中
  token: useApp().authorization.token,
}
// 定义提交表单数据模型
const defaultForm = {
  id: '',
  name: '',
  logo: '',
}
const brand = ref(defaultForm)
const dialogVisible = ref(false)

// 显示添加品牌表单
const addShow = () => {
  brand.value = {}
  dialogVisible.value = true
}

//上传
const handleAvatarSuccess = response => {
  brand.value.logo = response.data
}

// 保存数据
const saveOrUpdate = () => {
  if (brand.value.id) {
    updateBrandData() // 调用更新函数
  } else {
    saveData() // 原有的保存逻辑
  }
}

//修改数据
const updateBrandData = async () => {
  await UpdateBrandById(brand.value) // 假定这是更新品牌的函数
  ElMessage.success('品牌信息更新成功')
  dialogVisible.value = false
  fetchData() // 刷新列表
}

//删除数据
const deleteBrand = id => {
  ElMessageBox.confirm('确定要删除这个品牌吗?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  })
    .then(async () => {
      await DeleteBrandById(id) // 假定这是删除品牌的函数
      ElMessage.success('品牌删除成功')
      fetchData() // 刷新列表
    })
    .catch(() => {
      ElMessage.info('已取消删除')
    })
}

// 新增
const saveData = async () => {
  await SaveBrand(brand.value)
  dialogVisible.value = false
  ElMessage.success('操作成功')
  fetchData()
}

//列表
// 定义表格数据模型
const list = ref([])

// 分页条数据模型
const total = ref(0)

//分页条数据模型
const pageParamsForm = {
  page: 1, // 页码
  limit: 10, // 每页记录数
}
const pageParams = ref(pageParamsForm)

// 钩子函数
onMounted(() => {
  fetchData()
})

//页面变化
const handleSizeChange = size => {
  pageParams.value.limit = size
  fetchData()
}
const handleCurrentChange = number => {
  pageParams.value.page = number
  fetchData()
}

// 分页查询
const fetchData = async () => {
  const { code, message, data } = await GetBrandPageList(
    pageParams.value.page,
    pageParams.value.limit
  )
  list.value = data.list
  total.value = data.total
}
</script>

<style scoped>
.tools-div {
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ebeef5;
  border-radius: 3px;
  background-color: #fff;
}
</style>