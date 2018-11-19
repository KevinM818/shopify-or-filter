<template>
  <div id="app">
    <div class="ProductContainer">
      <div class="ProductContainer__filter">
        
      </div>
      <div class="ProductContainer__prods">
        <a class="ProductContainer__prod" v-for="product in displayedProducts" :href="'products/'+product.url">
          <img :src="product.img">
          <h3>{{ product.title }}</h3>
        </a>
      </div>
    </div>
    <div class="Pagination" v-if="pages.length > 1">
      <button v-if="page != 1" @click="page--"><<</button>
      <span v-if="pages.length > 6">
        <span v-for="pageNumber, index in pages">
          <button @click="page = pageNumber" v-if="(pageNumber >= page -2 && pageNumber <= page + 2) || pageNumber === 1 || pageNumber === pages[pages.length - 1]">{{ pageNumber }}</button>
          <span v-if="(page - 2 > 2 && index === 0) || (page + 2 < pages[pages.length - 2] && pageNumber === page + 2)">...</span>
        </span>
      </span>
      <span v-else>
        <span  v-for="pageNum in pages">
          <button @click="page = pageNum">{{ pageNum }}</button>
        </span>
      </span>
      <button @click="page++" v-if="page < pages.length" >>></button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'app',
  data() {
    return {
      allProducts: [],
      filterOptions: {},
      page: 1,
      perPage: 24,
      pages: []
    };
  },
  methods: {
    fetchProducts(pageNum) {
      axios.get(`${window.location.href}/products.json?limit=250&page=${pageNum}`)
        .then(response => {
          response.data.products.forEach(prod => this.getProdData({
            id: prod.id,
            title: prod.title,
            url: prod.handle,
            img: prod.images[0].src,
            tags: prod.tags.filter(tag => tag.includes('_'))
          }));
          if (response.data.products.length === 250) {
            this.fetchProducts(pageNum + 1);
          } else {
            this.setPages();
          }
        })
        .catch(error => console.log('Error fetching content', error));
    },
    getProdData(prod) {
      this.allProducts.push(prod);
      prod.tags.forEach(tag => {
        let filterKey = tag.split('_')[0].toLowerCase();
        let filterValue = tag.split('_')[1].toLowerCase();
        if (!Array.isArray(this.filterOptions[filterKey])) {
          this.filterOptions[filterKey] = [{
            option: filterValue,
            amount: 1
          }];
        } else {
          let updateIndex = this.filterOptions[filterKey].findIndex(val => val.option === filterValue);
          if (updateIndex === -1) {
            this.filterOptions[filterKey].push({
              option: filterValue,
              amount: 1
            });
          } else {
            this.filterOptions[filterKey][updateIndex].amount++;
          };
        };
      });
    },
    setPages() {
      let numberOfPages = Math.ceil(this.allProducts.length / this.perPage);
      this.pages = Array.from(Array(numberOfPages)).map((e,i)=>i+1)
    },
    paginate(prods) {
      let page = this.page;
      let perPage = this.perPage;
      let from = (page * perPage) - perPage;
      let to = (page * perPage);
      return  prods.slice(from, to);
    }
  },
  computed: {
    displayedProducts() {
      return this.paginate(this.allProducts);
    }
  },
  created() {
    this.fetchProducts(1);
  }
}
</script>

<style type="text/css" scoped>
 #app {
  width: 100%;
  max-width: 780px;
  margin: 0 auto 150px;
 }

 .ProductContainer__prods {
  display: flex;
  flex-flow: row wrap;
 }

 .ProductContainer__prod {
  width: 25%;
  padding: 10px;
  box-sizing: border-box;
 }

 img {
  max-width: 100%;
 }
</style>