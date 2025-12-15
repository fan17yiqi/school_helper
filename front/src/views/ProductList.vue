<template>
  <div class="product-list">
    <el-container>
      <el-header>
        <div class="header-content">
          <h2>商品列表</h2>
          <el-button type="primary" @click="$router.push('/product/publish')" icon="el-icon-plus">发布商品</el-button>
        </div>
      </el-header>
      <el-main>
        <el-row :gutter="20">
          <el-col :span="6">
            <el-card>
              <h3>分类筛选</h3>
              <el-radio-group v-model="selectedCategory" @change="loadProducts">
                <el-radio label="">全部</el-radio>
                <el-radio label="二手书">二手书</el-radio>
                <el-radio label="生活用品">生活用品</el-radio>
                <el-radio label="历年卷">历年卷</el-radio>
              </el-radio-group>
            </el-card>
          </el-col>
          <el-col :span="18">
            <el-row :gutter="20">
              <el-col :span="8" v-for="product in products" :key="product.id" style="margin-bottom: 20px;">
                <el-card class="product-card" @click.native="$router.push(`/product/${product.id}`)">
                  <div class="product-image">
                    <img v-if="getFirstImage(product.images)" :src="getFirstImage(product.images)" alt="商品图片">
                    <div v-else class="no-image">暂无图片</div>
                  </div>
                  <div class="product-info">
                    <h3>{{ product.title }}</h3>
                    <p class="price">¥{{ product.price }}</p>
                    <p class="category">{{ product.category }}</p>
                    <p class="trade-type">{{ product.tradeType === '自提' ? '自提' : '送达' }}</p>
                  </div>
                </el-card>
              </el-col>
            </el-row>
            <el-empty v-if="products.length === 0" description="暂无商品"></el-empty>
          </el-col>
        </el-row>
      </el-main>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'ProductList',
  data() {
    return {
      products: [],
      selectedCategory: ''
    }
  },
  mounted() {
    this.loadProducts()
  },
  methods: {
    loadProducts() {
      const params = this.selectedCategory ? { category: this.selectedCategory } : {}
      this.$http.get('/product/list', { params }).then(res => {
        if (res.success) {
          this.products = res.data
        }
      })
    },
    getFirstImage(images) {
      if (!images) return null
      try {
        const imgArray = JSON.parse(images)
        if (imgArray.length > 0) {
          let imgUrl = imgArray[0]
          // 确保URL是字符串且不为空
          if (!imgUrl || typeof imgUrl !== 'string') {
            return null
          }
          // 如果URL是相对路径，确保以/开头
          if (!imgUrl.startsWith('http') && !imgUrl.startsWith('/')) {
            imgUrl = '/' + imgUrl
          }
          // 如果是 /uploads/ 开头的路径，在开发环境中使用后端地址
          if (imgUrl.startsWith('/uploads/') && process.env.NODE_ENV === 'development') {
            imgUrl = 'http://47.108.59.70:8090' + imgUrl
          }
          return imgUrl
        }
        return null
      } catch {
        return null
      }
    }
  }
}
</script>

<style scoped>
.product-list {
  min-height: 100vh;
  background: #f5f5f5;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 100%;
  padding: 0 20px;
}

.product-card {
  cursor: pointer;
  transition: all 0.3s;
}

.product-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.product-image {
  width: 100%;
  height: 200px;
  overflow: hidden;
  margin-bottom: 10px;
}

.product-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.no-image {
  width: 100%;
  height: 100%;
  background: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
}

.product-info h3 {
  margin: 10px 0;
  font-size: 16px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.price {
  color: #f56c6c;
  font-size: 20px;
  font-weight: bold;
  margin: 10px 0;
}

.category, .trade-type {
  color: #666;
  font-size: 12px;
  margin: 5px 0;
}
</style>

