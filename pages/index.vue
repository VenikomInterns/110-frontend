<template>
  <main class="d-flex flex-column align-items-center bg-light bg-opacity-75 h-100">
    <div class="d-inline">
      <div>
      <span v-for="category in categoriesPerPage" :key="category.id" class="fs-4 btn"
            @click="getCategoryProducts(category.id)">
        {{ category.name }}
      </span>
      </div>
      <div class="my-3 d-flex flex-column align-items-center">
        <div v-for="(product, index) in productsArray" :key="product.id" class="mx-3">
          <i @click="storeShoppingCartToken(product.id, index)" class="fa fa-heart fa-2x btn py-0 px-3"
             aria-hidden="true"></i>
          <span class="btn bg-danger bg-opacity-25 my-3" @click="getProductInfo(product.id)">{{ product.name }}</span>
        </div>
      </div>
    </div>
    <div class="bg-opacity-25 bg-danger rounded rounded-3">
      <div class="p-4 d-flex flex-column align-items-center text-start fs-5 fw-bold" v-if="productName !== ''">
        <div class="mt-4 mb-2">
          NAME: {{ productName }}
        </div>
        <div class="my-2">
          PRICE: <span class="ps-2">$ {{ productPrice }}</span>
        </div>
        <div class="mt-2 mb-5 fs-6 fw-normal">
          DESCRIPTION: {{ productDescription }}
        </div>
        <div class="my-2 row">
          <img class="p-3 shadow col-3" v-for="image in imagesArray"
               :src="'http://127.0.0.1:8000/storage/images/' + image.image"
               alt="Image Alt" width="400" height="400">
        </div>
      </div>
    </div>
    <div class="my-5">
      <button @click="retrieveShoppingCartToken()" class="btn btn-dark text-white py-2 px-3">MY SHOPPING CART</button>
    </div>
    <div class="d-flex flex-row justify-content-between">
      <div class="my-4" v-if="isClicked" v-for="shoppingCartProduct in cartResult">
        <div v-if="shoppingCartProduct !== null" class="fw-bold fs-5">
          <div class="mt-4 mb-2">
            NAME: {{ shoppingCartProduct.name }}
          </div>
          <div class="my-2">
            PRICE: <span class="ps-2">$ {{ shoppingCartProduct.price }}</span>
          </div>
          <div class="mt-2 mb-5 fs-6 fw-normal col-6">
            DESCRIPTION: {{ shoppingCartProduct.description }}
          </div>
        </div>
      </div>
    </div>
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li class="page-item" v-for="page in pagesNumber">
          <button class="page-link text-danger" @click="selectedPageProducts(page)">{{ page }}</button>
        </li>
      </ul>
    </nav>
  </main>
</template>

<script>
  // I don't like putting everything in one page and one file.
  // This code looks so much complex 
  // Why you always call JSON.parse(JSON.stringify()) hmmmm 
  
export default {
  name: 'IndexPage',
  data() {
    return {
      categories: [],
      productsInSelectedCategory: [],
      productsArray: [],
      infoForSelectedProduct: [],
      imagesArray: [],
      productName: '',
      productPrice: 0,
      productDescription: '',
      shoppingCartProducts: [],
      isClicked: false,
      cartResult: [],
      pagesNumber: 0,
      clickedPage: 1,
      productsPerPage: [],
      categoriesPerPage: [],
    }
  },
  created() {
    this.$axios.$get(`/api/categories?page=${this.clickedPage}`)
      .then((response) => {
        this.productsPerPage = JSON.parse(JSON.stringify(response.meta));
        this.categoriesPerPage = JSON.parse(JSON.stringify(response.data));
        this.clickedPage = this.productsPerPage.current_page;
        this.pagesNumber = this.productsPerPage.last_page;
      })
      .catch((error) => {
        console.log(error);
      })
  },
  methods: {
    getCategoryProducts(index) {
      this.$axios.get('/api/categories/' + index)
        .then((response) => {
          this.productsInSelectedCategory = JSON.parse(JSON.stringify(response.data));
          this.productsArray = this.productsInSelectedCategory.data.products;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getProductInfo(index) {
      this.$axios.get('/api/products/' + index)
        .then((response) => {
          this.infoForSelectedProduct = JSON.parse(JSON.stringify(response.data));
          this.imagesArray = this.infoForSelectedProduct.data.images;
          this.productName = this.infoForSelectedProduct.data.name;
          this.productPrice = this.infoForSelectedProduct.data.price;
          this.productDescription = this.infoForSelectedProduct.data.description;
        })
        .catch((error) => {
          console.log(error);
        })
    },
    storeShoppingCartToken(token, index) {
      if (this.shoppingCartProducts[token] != null) {
        this.shoppingCartProducts[token] = null;
      } else {
        this.shoppingCartProducts[token] = this.productsArray[index];
      }
      localStorage.setItem('shoppingCartToken', JSON.stringify(this.shoppingCartProducts));
    },
    retrieveShoppingCartToken() {
      console.log(JSON.parse(localStorage.getItem('shoppingCartToken')));
      this.cartResult = JSON.parse(localStorage.getItem('shoppingCartToken'));
      this.isClicked = true;
    },
    selectedPageProducts(page) {
      this.$axios.$get(`/api/categories?page=${page}`)
        .then((response) => {
          this.productsPerPage = JSON.parse(JSON.stringify(response.meta));
          this.categoriesPerPage = JSON.parse(JSON.stringify(response.data));
          this.clickedPage = this.productsPerPage.current_page;
          this.pagesNumber = this.productsPerPage.last_page;
        })
        .catch((error) => {
          console.log(error);
        })
    }
  }
}
</script>
