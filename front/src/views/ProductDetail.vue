<template>
  <Layout>
    <div class="product-detail-page" v-if="product">
      <el-row :gutter="20">
        <!-- 左侧：商品图片 -->
        <el-col :span="12">
          <el-card>
            <el-carousel v-if="productImages.length > 0" height="500px">
              <el-carousel-item v-for="(img, index) in productImages" :key="index">
                <img :src="img" style="width: 100%; height: 100%; object-fit: contain;">
              </el-carousel-item>
            </el-carousel>
            <div v-else class="no-image-large">
              <i class="el-icon-picture"></i>
              <p>暂无图片</p>
            </div>
          </el-card>
        </el-col>

        <!-- 右侧：商品信息 -->
        <el-col :span="12">
          <el-card>
            <h1 class="product-title">{{ product.title }}</h1>
            <div class="product-price-large">¥{{ product.price }}</div>
            
            <el-divider></el-divider>

            <div class="info-row">
              <span class="info-label">分类：</span>
              <el-tag>{{ product.category }}</el-tag>
            </div>
            <div class="info-row">
              <span class="info-label">交易方式：</span>
              <el-tag :type="product.tradeType === '自提' ? 'success' : 'warning'">
                {{ product.tradeType }}
              </el-tag>
            </div>
            <div class="info-row" v-if="product.tradeType === '送达' && deliveryPoint">
              <span class="info-label">送达地点：</span>
              <span>{{ deliveryPoint.name }} - {{ deliveryPoint.address }}</span>
            </div>
            <div class="info-row">
              <span class="info-label">状态：</span>
              <el-tag :type="product.status === 0 ? 'success' : 'info'">
                {{ product.status === 0 ? '待售' : product.status === 1 ? '已售' : '已下架' }}
              </el-tag>
            </div>

            <el-divider></el-divider>

            <h3>商品描述</h3>
            <div class="product-description">{{ product.description }}</div>

            <el-divider></el-divider>

            <div class="action-buttons">
              <el-button 
                type="primary" 
                size="large" 
                icon="el-icon-message"
                @click="contactSeller"
                :disabled="product.status !== 0 || isOwner">
                {{ isOwner ? '这是您发布的商品' : product.status === 0 ? '联系卖家' : '商品已售' }}
              </el-button>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>
  </Layout>
</template>

<script>
import Layout from '../components/Layout.vue'

export default {
  name: 'ProductDetail',
  components: {
    Layout
  },
  data() {
    return {
      product: null,
      deliveryPoint: null,
      productImages: [],
      user: null
    }
  },
  computed: {
    isOwner() {
      return this.user && this.product && this.user.id === this.product.userId
    }
  },
  mounted() {
    this.user = JSON.parse(localStorage.getItem('user') || 'null')
    this.loadProduct()
  },
  methods: {
    loadProduct() {
      const id = this.$route.params.id
      if (!id || id === 'publish') {
        this.$message.error('商品ID无效')
        this.$router.push('/')
        return
      }
      this.$http.get(`/product/${id}`).then(res => {
        if (res.success) {
          this.product = res.data
          if (this.product.images) {
            try {
              const imgArray = JSON.parse(this.product.images)
              // 处理图片URL，确保可以正确访问
              this.productImages = imgArray
                .filter(img => img && typeof img === 'string' && img.trim() !== '')
                .map(img => {
                  // 如果URL是相对路径，确保以/开头
                  if (!img.startsWith('http') && !img.startsWith('/')) {
                    img = '/' + img
                  }
                  // 如果是 /uploads/ 开头的路径，在开发环境中使用后端地址
                  if (img.startsWith('/uploads/') && process.env.NODE_ENV === 'development') {
                    img = 'http://47.108.59.70:8090' + img
                  }
                  return img
                })
            } catch {
              this.productImages = []
            }
          }
          if (this.product.deliveryPointId) {
            this.loadDeliveryPoint(this.product.deliveryPointId)
          }
        }
      })
    },
    loadDeliveryPoint(id) {
      this.$http.get(`/delivery/${id}`).then(res => {
        if (res.success) {
          this.deliveryPoint = res.data
        }
      })
    },
    contactSeller() {
      if (!this.user) {
        this.$message.warning('请先登录')
        this.$router.push('/login')
        return
      }
      // 跳转到聊天页面，带上商品ID
      this.$router.push({
        path: `/chat/${this.product.userId}`,
        query: { productId: this.product.id }
      })
    }
  }
}
</script>

<style scoped>
.product-detail-page {
  width: 100%;
}

.product-title {
  font-size: 28px;
  margin-bottom: 20px;
  color: #333;
}

.product-price-large {
  font-size: 36px;
  color: #f56c6c;
  font-weight: bold;
  margin-bottom: 20px;
}

.info-row {
  margin: 15px 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.info-label {
  font-weight: bold;
  color: #666;
  min-width: 80px;
}

.product-description {
  margin: 20px 0;
  line-height: 1.8;
  color: #333;
  white-space: pre-wrap;
}

.action-buttons {
  margin-top: 30px;
}

.action-buttons .el-button {
  width: 100%;
  padding: 15px;
  font-size: 16px;
}

.no-image-large {
  height: 500px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #999;
}

.no-image-large i {
  font-size: 80px;
  margin-bottom: 20px;
}
</style>
