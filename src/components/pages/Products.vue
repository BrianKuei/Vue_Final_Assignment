<template>
  <div>
    <div class="vld-parent">
        <loading :active.sync="isLoading"></loading>
    </div>
    <div class="text-right mt-4">
      <button class="btn btn-primary" @click="openModal(true)">建立新的產品</button>
    </div>
    <table class="table mt-4">
      <thead>
        <tr>
          <th width="120">分類</th>
          <th>產品名稱 </th>
          <th width="100">原價</th>
          <th width="100">售價</th>
          <th width="100">是否啟用</th>
          <th width="120">編輯</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in products" :key="item.id">
          <td>{{item.category}}</td>
          <td>{{item.title}}</td>
          <td>{{item.origin_price | currency}}</td>
          <td>{{item.price | currency}}</td>
          <td>
            <span v-if="item.is_enabled" class="text-success">啟用</span>
            <span v-else>未啟用</span>
          </td>
          <td>
            <button class="btn btn-outline-primary btn-sm" 
                @click="openModal(false, item)">編輯</button>
            <button class="btn btn-outline-danger btn-sm" 
                @click="openDeletModal(item)">刪除</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- pagination -->
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li class="page-item" :class="{'disabled': !pagination.has_pre}"
            @click.prevent="getProducts(pagination.current_page - 1)">
          <a class="page-link" href="#" aria-label="Previous">
            <span aria-hidden="true">&laquo;</span>
          </a>
        </li>
        <li class="page-item" v-for="page in pagination.total_pages" :key="page"
            :class="{'active': pagination.current_page === page}">
          <a class="page-link" href="#" @click.prevent="getProducts(page)">{{page}}</a>
        </li>
        <li class="page-item" :class="{'disabled': !pagination.has_next}"
            @click.prevent="getProducts(pagination.current_page + 1)">
          <a class="page-link" href="#" aria-label="Next">
            <span aria-hidden="true">&raquo;</span>
          </a>
        </li>
      </ul>
    </nav>

    <!-- Modal -->
    <!-- 新增的 Modal -->
    <div class="modal fade" id="productModal" tabindex="-1" role="dialog"
      aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-lg" role="document">
        <div class="modal-content border-0">
          <div class="modal-header bg-dark text-white">
            <h5 class="modal-title" id="exampleModalLabel">
              <span>新增產品</span>
            </h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="row">
              <div class="col-sm-4">
                <div class="form-group">
                  <label for="image">輸入圖片網址</label>
                  <input type="text" class="form-control" id="image"
                    v-model="tempProduct.imageUrl"
                    placeholder="請輸入圖片連結">
                </div>
                <div class="form-group">
                  <label for="customFile">或 上傳圖片
                    <i class="fas fa-spinner fa-spin" v-if="status.fileUploading"></i>
                  </label>
                  <input type="file" id="customFile" class="form-control"
                    @change="uploadFile"
                    ref="files">
                </div>
                <img img="https://images.unsplash.com/photo-1483985988355-763728e1935b?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=828346ed697837ce808cae68d3ddc3cf&auto=format&fit=crop&w=1350&q=80"
                  class="img-fluid" :src="tempProduct.imageUrl" alt="">
              </div>
              <div class="col-sm-8">
                <div class="form-group">
                  <label for="title">標題</label>
                  <input type="text" class="form-control" id="title"
                    v-model="tempProduct.title"
                    placeholder="請輸入標題">
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                    <label for="category">分類</label>
                    <input type="text" class="form-control" id="category"
                      v-model="tempProduct.category"
                      placeholder="請輸入分類">
                  </div>
                  <div class="form-group col-md-6">
                    <label for="unit">單位</label>
                    <input type="unit" class="form-control" id="unit"
                      v-model="tempProduct.unit"
                      placeholder="請輸入單位">
                  </div>
                </div>

                <div class="form-row">
                  <div class="form-group col-md-6">
                  <label for="origin_price">原價</label>
                    <input type="number" class="form-control" id="origin_price"
                      v-model="tempProduct.origin_price"
                      placeholder="請輸入原價">
                  </div>
                  <div class="form-group col-md-6">
                    <label for="price">售價</label>
                    <input type="number" class="form-control" id="price"
                      v-model="tempProduct.price"
                      placeholder="請輸入售價">
                  </div>
                </div>
                <hr>

                <div class="form-group">
                  <label for="description">產品描述</label>
                  <textarea type="text" class="form-control" id="description"
                    v-model="tempProduct.description"
                    placeholder="請輸入產品描述"></textarea>
                </div>
                <div class="form-group">
                  <label for="content">說明內容</label>
                  <textarea type="text" class="form-control" id="content"
                    v-model="tempProduct.content"
                    placeholder="請輸入產品說明內容"></textarea>
                </div>
                <div class="form-group">
                  <div class="form-check">
                    <input class="form-check-input" type="checkbox"
                      v-model="tempProduct.is_enabled"
                      :true-value="1"
                      :false-value="0"
                      id="is_enabled">
                    <label class="form-check-label" for="is_enabled">
                      是否啟用
                    </label>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">取消</button>
            <button type="button" class="btn btn-primary" @click="updateProduct">確認</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 刪除的 Modal -->
    <div class="modal fade" id="delProductModal" tabindex="-1" role="dialog"
      aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog" role="document">
        <div class="modal-content border-0">
          <div class="modal-header bg-danger text-white">
            <h5 class="modal-title" id="exampleModalLabel">
              <span>刪除產品</span>
            </h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            是否刪除 <strong class="text-danger">{{ tempProduct.title }}</strong> 商品(刪除後將無法恢復)。
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-outline-secondary" data-dismiss="modal">取消</button>
            <button type="button" class="btn btn-danger"
              @click="deleteProduct"
              >確認刪除</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import $ from 'jquery';

export default {
  data() {
    return {
      products:[],
      pagination:{},
      tempProduct:{},
      isNew: false,
      isLoading:false,
      status: {
        fileUploading:false,
      }
    }
  },
  methods: {
    /* 這裏使用 es6 的參數預設，假設沒有帶數值進來預設就會是在第一頁，如果有其他數值就會帶過來
    ，這樣其他有使用到 getProducts() 的地方就不用補上預設值了。 */
    getProducts(page = 1){ 
      const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/products?page=${page}`;
      const vm = this;
      vm.isLoading = true; 
      this.$http.get(api).then((response) => {
        console.log(response.data);
        vm.isLoading = false; 
        vm.products = response.data.products;
        vm.pagination = response.data.pagination;
      });
    },
    openModal(isNew, item){
      if (isNew){
        this.tempProduct = {};
        this.isNew = true;
      } else {

        // item 存的是我們點選的物件值
        /* 因為物件是傳參考，所以用 this.tempProduct = item 這兩個值就會一樣
        ，所以用 es6 Object.assign() 把新的值寫道空的物件中，這樣就可以避免 item 有參考的特性 */
        this.tempProduct = Object.assign({}, item);
        this.isNew = false;
      }
      // $ 要載入 jquery
      $('#productModal').modal('show'); 
    },
    openDeletModal(item){
      this.tempProduct = item;
      $('#delProductModal').modal('show'); 
    },
    uploadFile(){
      const uploadedFile = this.$refs.files.files[0];
      const vm = this;
      vm.status.fileUploading = true;
      // 可以用 formdata 來模擬傳統表單送出的形式 
      const formData = new FormData();
      formData.append('uploadedFile',uploadedFile);
      const url = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/upload`;
      this.$http.post(url, formData, {
        headers: {
          'Content-Type': ' multipart/form-data',
        }
      }).then((response) => {
        console.log(response.data);
        vm.status.fileUploading = false ;
        if (response.data.success){
          // vm.tempProduct.imageUrl = response.data.imageUrl;
          vm.$set(vm.tempProduct, 'imageUrl', response.data.imageUrl);
        } else {
          this.$bus.$emit('message:push', response.data.message, 'danger');
        }
      })
    },
    updateProduct(){
      let api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/product`;
      let httpMethod = 'post';
      const vm = this;
      if (!vm.isNew){
        api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/product/${vm.tempProduct.id}`;
        httpMethod = 'put';
      }
      this.$http[httpMethod](api, { data: vm.tempProduct }).then((response) => {
        console.log(response.data);
        if (response.data.success){
          $('#productModal').modal('hide');
          vm.getProducts();
        } else  {
          $('#productModal').modal('hide');
          vm.getProducts();
          this.$bus.$emit('message:push', '新增失敗', 'danger')
        }
      });
    },
    deleteProduct(){
      const vm = this;
      const api = `${process.env.APIPATH}/api/${process.env.CUSTOMPATH}/admin/product/${vm.tempProduct.id}`;
      vm.$http.delete(api).then((response) => {
        console.log(response.data);
        if (response.data.success){
          $('#delProductModal').modal('hide');
          vm.getProducts();
          console.log(response.data.message);
        } else  {
          $('#delProductModal').modal('hide');
          vm.getProducts();
          console.log(response.data.message);
        }
      });
    }
  },
  created() {
    this. getProducts();
  },
}
</script>