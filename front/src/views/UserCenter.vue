<template>
  <Layout>
    <div class="user-center">
      <el-card>
        <div slot="header">
          <h2>个人中心</h2>
        </div>
        <el-tabs>
          <el-tab-pane label="我的商品">
            <el-table :data="myProducts" style="width: 100%">
              <el-table-column prop="title" label="标题" width="200"></el-table-column>
              <el-table-column prop="category" label="分类" width="120"></el-table-column>
              <el-table-column prop="price" label="价格" width="100"></el-table-column>
              <el-table-column prop="status" label="状态" width="100">
                <template slot-scope="scope">
                  <el-tag :type="scope.row.status === 0 ? 'success' : 'info'">
                    {{ scope.row.status === 0 ? '待售' : scope.row.status === 1 ? '已售' : '已下架' }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button size="small" @click="editProduct(scope.row)">编辑</el-button>
                  <el-button size="small" type="danger" @click="deleteProduct(scope.row.id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="我的求助">
            <el-table :data="myHelps" style="width: 100%">
              <el-table-column prop="title" label="标题" width="200"></el-table-column>
              <el-table-column prop="category" label="分类" width="120"></el-table-column>
              <el-table-column prop="status" label="状态" width="100">
                <template slot-scope="scope">
                  <el-tag :type="scope.row.status === 0 ? 'success' : 'info'">
                    {{ scope.row.status === 0 ? '进行中' : scope.row.status === 1 ? '已完成' : '已取消' }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button size="small" @click="editHelp(scope.row)">编辑</el-button>
                  <el-button size="small" type="danger" @click="deleteHelp(scope.row.id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
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
  name: 'UserCenter',
  data() {
    return {
      myProducts: [],
      myHelps: []
    }
  },
  mounted() {
    const user = JSON.parse(localStorage.getItem('user') || 'null')
    if (!user) {
      this.$message.warning('请先登录')
      this.$router.push('/login')
      return
    }
    this.loadMyProducts(user.id)
    this.loadMyHelps(user.id)
  },
  methods: {
    loadMyProducts(userId) {
      this.$http.get(`/product/user/${userId}`).then(res => {
        if (res.success) {
          this.myProducts = res.data
        }
      })
    },
    loadMyHelps(userId) {
      this.$http.get(`/help/user/${userId}`).then(res => {
        if (res.success) {
          this.myHelps = res.data
        }
      })
    },
    editProduct(product) {
      // 编辑商品逻辑
      this.$message.info('编辑功能待实现')
    },
    deleteProduct(id) {
      this.$confirm('确定要删除这个商品吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete(`/product/${id}`).then(res => {
          if (res.success) {
            this.$message.success('删除成功')
            const user = JSON.parse(localStorage.getItem('user') || 'null')
            this.loadMyProducts(user.id)
          }
        })
      })
    },
    editHelp(help) {
      // 编辑求助逻辑
      this.$message.info('编辑功能待实现')
    },
    deleteHelp(id) {
      this.$confirm('确定要删除这个求助吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete(`/help/${id}`).then(res => {
          if (res.success) {
            this.$message.success('删除成功')
            const user = JSON.parse(localStorage.getItem('user') || 'null')
            this.loadMyHelps(user.id)
          }
        })
      })
    }
  }
}
</script>

<style scoped>
.user-center {
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

