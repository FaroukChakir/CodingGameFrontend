<template>
  <div>
    <h1>Product List</h1>
    
    <!-- Search input -->
    <input type="text" v-model="search" placeholder="Search by category...">
    <button @click="searchProducts">Search</button>
    
    <!-- Display products or message based on search result -->
    <div v-if="categoryFound">
      <!-- Product list -->
      <ul>
        <li v-for="product in products" :key="product.id">
          {{ product.name }} - ${{ product.price }}
        </li>
      </ul>
      
      <!-- Pagination -->
      <button @click="previousPage" :disabled="currentPage === 1">Previous</button>
      Page {{ currentPage }} of {{ totalPages }}
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>
    <div v-else>
      <!-- Display a message when category is not found -->
      No category found.
    </div>
  </div>
</template>

<script>
import axios from 'axios';


export default {
  data() {
    return {
      products: [],
      currentPage: 1,
      itemsPerPage: 10,
      search: '',
      totalPages: 0,
      categoryFound: true, 
    };
  },
  methods: {
    async fetchData(page = 1) {
      try {
        const response = await axios.get(`http://127.0.0.1:8000/api/GetProducts`, {
          params: {
            per_page: this.itemsPerPage,
            page: page,
          },
        });
        this.products = response.data.data;
        this.totalPages = response.data.last_page;
        this.currentPage = response.data.current_page;
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.fetchData(this.currentPage);
      }
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.fetchData(this.currentPage);
      }
    },
    async searchProducts() {
      try {
        const response = await axios.get(`http://127.0.0.1:8000/api/SearchProductsByCategory`, {
          params: {
            category: this.search,
          },
        });

        if (response.data.message) {
          // CATegory not found
          this.categoryFound = false; 
        } else {
          // catego ry was found
          this.categoryFound = true; // Set categoryFound to true
          this.products = response.data.data;
          this.totalPages = response.data.last_page;
          this.currentPage = response.data.current_page;
        }
      } catch (error) {
        console.error('Error searching products:', error);
      }
    },
  },
  created() {
    this.fetchData();
  },
};
</script>
