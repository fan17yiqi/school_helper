<template>
  <Layout>
    <div class="help-publish">
      <el-card>
        <div slot="header">
          <h2>发布求助</h2>
        </div>
        <el-form :model="helpForm" :rules="rules" ref="helpForm" label-width="120px" style="max-width: 800px;">
          <el-form-item label="求助标题" prop="title">
            <el-input v-model="helpForm.title" placeholder="请输入求助标题"></el-input>
          </el-form-item>
          <el-form-item label="求助分类" prop="category">
            <el-select v-model="helpForm.category" placeholder="请选择分类">
              <el-option label="拼车服务" value="拼车服务"></el-option>
              <el-option label="快递代取" value="快递代取"></el-option>
              <el-option label="求购服务" value="求购服务"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="求助描述" prop="description">
            <el-input type="textarea" :rows="5" v-model="helpForm.description" placeholder="请输入详细描述"></el-input>
          </el-form-item>
          <el-form-item label="相关图片">
            <el-upload
              action="/api/file/upload"
              list-type="picture-card"
              :file-list="fileList"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :on-success="handleUploadSuccess"
              :on-error="handleUploadError"
              :before-upload="beforeUpload"
              :limit="5">
              <i class="el-icon-plus"></i>
            </el-upload>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="handleSubmit">发布</el-button>
            <el-button @click="$router.back()">取消</el-button>
          </el-form-item>
        </el-form>
      </el-card>
    </div>
  </Layout>
</template>

<script>
import Layout from '../components/Layout.vue'

export default {
  components: {
    Layout
  },
  name: 'HelpPublish',
  data() {
    return {
      helpForm: {
        title: '',
        category: '',
        description: ''
      },
      rules: {
        title: [{ required: true, message: '请输入求助标题', trigger: 'blur' }],
        category: [{ required: true, message: '请选择分类', trigger: 'change' }],
        description: [{ required: true, message: '请输入求助描述', trigger: 'blur' }]
      },
      fileList: []
    }
  },
  mounted() {
    const user = JSON.parse(localStorage.getItem('user') || 'null')
    if (!user) {
      this.$message.warning('请先登录')
      this.$router.push('/login')
    }
  },
  methods: {
    handlePreview(file) {
      // 预览图片
      window.open(file.url || file.response?.url, '_blank')
    },
    handleRemove(file, fileList) {
      this.fileList = fileList
    },
    beforeUpload(file) {
      const isImage = file.type.startsWith('image/')
      const isLt5M = file.size / 1024 / 1024 < 5

      if (!isImage) {
        this.$message.error('只能上传图片文件!')
        return false
      }
      if (!isLt5M) {
        this.$message.error('图片大小不能超过 5MB!')
        return false
      }
      return true
    },
    handleUploadSuccess(response, file, fileList) {
      if (response.success) {
        // 确保保存服务器返回的真实URL，而不是blob URL
        // Element UI 可能会生成 blob URL 用于预览，我们需要强制使用服务器返回的URL
        let serverUrl = response.url
        // 如果是 /uploads/ 开头的路径，在开发环境中使用后端地址用于预览
        if (serverUrl.startsWith('/uploads/') && process.env.NODE_ENV === 'development') {
          serverUrl = 'http://47.108.59.70:8090' + serverUrl
        }
        this.fileList = fileList.map(f => {
          if (f.uid === file.uid) {
            return {
              ...f,
              url: serverUrl, // 使用服务器返回的URL（开发环境已转换为完整URL）
              response: response // 保存完整的响应，包含原始服务器URL
            }
          }
          return f
        })
        this.$message.success('图片上传成功')
      } else {
        this.$message.error(response.message || '上传失败')
        // 移除失败的文件
        this.fileList = fileList.filter(f => f.uid !== file.uid)
      }
    },
    handleUploadError(err, file, fileList) {
      this.$message.error('图片上传失败：' + (err.message || '网络错误'))
      // 移除失败的文件
      this.fileList = fileList.filter(f => f.uid !== file.uid)
    },
    handleSubmit() {
      this.$refs.helpForm.validate((valid) => {
        if (valid) {
          const user = JSON.parse(localStorage.getItem('user') || 'null')
          // 提取所有已上传图片的URL
          // 优先使用 response.url（服务器返回的真实URL），避免使用 blob URL 和完整URL
          const imageUrls = this.fileList
            .map(f => {
              // 优先使用服务器返回的原始URL（相对路径），用于保存到数据库
              const serverUrl = f.response?.url
              if (serverUrl && !serverUrl.startsWith('blob:') && !serverUrl.startsWith('http://')) {
                return serverUrl
              }
              // 如果 response.url 是完整URL，提取相对路径部分
              if (serverUrl && serverUrl.startsWith('http://47.108.59.70:8090/uploads/')) {
                return serverUrl.replace('http://47.108.59.70:8090', '')
              }
              // 如果 file.url 是完整URL，提取相对路径部分
              const fileUrl = f.url
              if (fileUrl && fileUrl.startsWith('http://47.108.59.70:8090/uploads/')) {
                return fileUrl.replace('http://47.108.59.70:8090', '')
              }
              // 如果 file.url 是相对路径且不是 blob URL
              if (fileUrl && !fileUrl.startsWith('blob:') && !fileUrl.startsWith('http://')) {
                return fileUrl
              }
              return ''
            })
            .filter(url => url && url.trim() !== '')
          
          const formData = {
            ...this.helpForm,
            userId: user.id,
            images: JSON.stringify(imageUrls)
          }
          this.$http.post('/help/add', formData).then(res => {
            if (res.success) {
              this.$message.success('发布成功')
              this.$router.push('/helps')
            } else {
              this.$message.error(res.message)
            }
          })
        }
      })
    }
  }
}
</script>

<style scoped>
.help-publish {
  min-height: 100vh;
  background: #f5f5f5;
}

.el-header {
  background: #fff;
  display: flex;
  align-items: center;
  padding: 0 20px;
}

.el-main {
  padding: 20px;
  background: #fff;
  margin: 20px;
  border-radius: 4px;
}
</style>

