<script>
import Navbar from "@/components/Navbar.vue";
import Loading from "@/components/Loading.vue";

const API_BASE_URL = import.meta.env.VITE_BASE_API_URL;
const API_PATH = import.meta.env.VITE_API_PATH;

export default {
  components: {Loading, Navbar},
  data() {
    return {
      isLogin: -1,
      page: 'product',
      accessToken: /accessToken=([^;]+)/.exec(document.cookie) && /accessToken=([^;]+)/.exec(document.cookie)[1],
      products: [],
      productsMap: {},
      targetProduct: null,
      getProductsSuccess: 0,
    }
  },
  created() {
    if (!this.accessToken) {
      alert('請先登入!');
      window.location.href = '/';
    }
    this.$axios.defaults.headers.common.authorization = this.accessToken;
    this.getAllProducts();
    if (this.accessToken) {
      this.checkUserStatus();
    }
    this.products.forEach((item, index) => {
      this.productsMap[item.id] = item;
    })
  },
  methods: {
    getAllProducts() {
      this.$axios.get(`${API_BASE_URL}v2/api/${API_PATH}/admin/products/all`).then((res) => {
        this.productsMap = res.data.products;
        let productData = null;
        for (let key in this.productsMap) {
          productData = this.productsMap[key];
          productData['id'] = key;
          this.products.push(productData)
          this.getProductsSuccess = 1;
        }
      }).catch((err) => {
        console.log(err);
        this.getProductsSuccess = -1;
      })
    },
    displayProduct(productId) {
      this.targetProduct = this.productsMap[productId];
    },
    checkUserStatus() {
      this.isLogin = 0;
      this.$axios.post(`${API_BASE_URL}v2/api/user/check`).then(() => {
        this.isLogin = 1;
      }).catch((err) => {
        console.log(err);
        alert('請重新登入!')
        this.isLogin = -1;
        window.location.href = '/';
      })
    },
  }
}
</script>

<template>
  <Navbar :login-status=isLogin :page="page"></Navbar>
  <template v-if="getProductsSuccess === 1">
    <div class="container">
      <div class="row py-3">
        <div class="col-md-6" id="product-list">
          <h2>產品列表</h2>
          <table class="table table-hover mt-4">
            <thead>
            <tr>
              <th width="150">產品名稱</th>
              <th width="120">
                原價
              </th>
              <th width="120">
                售價
              </th>
              <th width="150">
                是否啟用
              </th>
              <th width="120">
                查看細節
              </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="product in products" :key="product.id">
              <td width="150">{{ product.title }}</td>
              <td width="120">
                {{ product.origin_price.toLocaleString() }}
              </td>
              <td width="120">
                {{ product.price.toLocaleString() }}
              </td>
              <td width="150">
                <template v-if="product.is_enabled !== 0">
                  <span class="text-success">啟用</span>
                </template>
                <template v-else>
                  <span>未啟用</span>
                </template>
              </td>
              <td width="120">
                <button type="button" class="btn btn-primary" @click="displayProduct(product.id)">查看細節</button>
              </td>
            </tr>
            </tbody>
          </table>
          <p>目前有 <span>{{ products.length }}</span> 項產品</p>
        </div>
        <div class="col-md-6">
          <h2>單一產品細節</h2>
          <template v-if="targetProduct">
            <div class="card mb-3">
              <img :src="targetProduct.imageUrl" class="card-img-top primary-image" alt="主圖">
              <div class="card-body">
                <h5 class="card-title">
                  {{ targetProduct.title }}
                  <span class="badge bg-primary ms-2">{{ targetProduct.category }}</span>
                </h5>
                <p class="card-text">商品描述：{{ targetProduct.description }}</p>
                <p class="card-text">商品內容：{{ targetProduct.content }}</p>
                <div class="d-flex">
                  <p class="card-text me-2">{{ }}</p>
                  <p class="card-text text-secondary">
                    <del>{{ targetProduct.origin_price.toLocaleString() }}</del>
                  </p>
                  元 / {{ targetProduct.price.toLocaleString() }} 元
                </div>
              </div>
            </div>
            <template v-for="(detailImageUrl, id) in targetProduct.imagesUrl" :key="id">
              <img :src="detailImageUrl" alt="" class="images m-2">
            </template>
          </template>
          <template v-else>
            <p class="text-secondary">請選擇一個商品查看</p>
          </template>
        </div>
      </div>
    </div>
  </template>
  <template v-else-if="getProductsSuccess === 0">
    <Loading />
  </template>
  <template v-else>
    API Server Error
  </template>

</template>
