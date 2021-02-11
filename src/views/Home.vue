<template>
  <div class="home">
    <h2>Add a new product</h2>
    <div class="new-product-form">
      <label class="new-product-form__field">
        <div>Name:</div>
        <input type="text" v-model="newName">
      </label>
      <br>
      <label class="new-product-form__field">
        <div>Description:</div>
        <input type="text" v-model="newDescription">
      </label>
      <br>
      <label class="new-product-form__field">
        <div>Price:</div>
        <input type="text" v-model="newPrice">
      </label>
      <br>
      <label class="new-product-form__field">
        <div>Image URL:</div>
        <input type="text" v-model="newImageUrl">
      </label>
      <br>
      <div v-for="error in errors" :key="error">
        <p class="error">{{ error }}</p>
      </div>
      <button v-on:click="createProduct">Create</button>
    </div>

    <ul class="product-list" v-for="product in products" v-bind:key="product.id">
      <li class="product">
        <h2 class="product__header">{{ product.name }}</h2>
        <p class="product__price">${{ product.price }}</p>
        <p class="product__description">Description: {{ product.description }}</p>
        <img class="product__img" :src="product.image_url" :alt="'Image of ' + product.name">
        <button v-on:click="showProduct(product)">More info</button>
      </li>
    </ul>

    <dialog>
      <form method="dialog">
        <h3>Product Info</h3>
        <img :src="currentProduct.image_url" :alt="currentProduct.name">
        <h4>Name: <input type="text" v-model="currentProduct.name"></h4>
        <p>Price: <input type="text" v-model="currentProduct.price"></p>
        <p>Description: <input type="text" v-model="currentProduct.description"></p>
        <p>Image URL: <input type="text" v-model="currentProduct.image_url"></p>
        <button v-on:click="updateProduct(currentProduct)">Update</button>
        <button v-on:click="destroyProduct(currentProduct)">Delete</button>
        <button>Close</button>
      </form>
    </dialog>
  </div>
</template>

<style>
  button {
    background-color: #42b983;
    border: 0;
    color: #fff;
    padding: 8px 12px;
    cursor: pointer;
    transition: background-color 0.5s ease-out;
  }

  button + button {
    margin-left: 10px;
  }

  button:hover,
  button:focus {
    background-color: #333;
  }

  .product-list {
    max-width: 400px;
    margin: 0 auto;
  }

  .product {
    list-style-type: none;
    margin-bottom: 100px;
    border: 1px solid #42b983;
    padding: 30px;
    box-shadow: 7px 5px 0 #42b983;
  }

  .product__header {
    float: left;
    margin-top: 0;
  }

  .product__price {
    float: right;
    margin-top: 0;
    font-weight: bold;
  }

  .product__description {
    clear: both;
    text-align: left;
  }

  .product__img,
  dialog img {
    width: 100%;
  }

  .new-product-form {
    text-align: left;
    width: 400px;
    margin: 0 auto 50px;
  }

  .new-product-form  button {
    width: 100%;
    padding: 10px;
  }

  .new-product-form__field {
    display: block;
    margin-bottom: 10px;
  }

  .new-product-form__field input {
    width: 100%;
    padding: 10px;
    box-sizing: border-box;
  }

  .error {
    color: red;
  }

  dialog {
    width: 400px;
  }
</style>

<script>
import axios from "axios";

export default {
  data: function() {
    return {
      products: [],
      newName: "",
      newDescription: "",
      newPrice: "",
      newImageUrl: "",
      errors: [],
      currentProduct: {},
    };
  },
  created: function() {
    this.indexProducts();
  },
  methods: {
    indexProducts: function() {
      axios.get("/api/products").then(response => {
        this.products = response.data;
      });
    },
    createProduct: function() {
      var params = {
        name: this.newName,
        description: this.newDescription,
        image_url: this.newImageUrl,
        price: this.newPrice,
      };

      axios
        .post("/api/products", params)
        .then(response => {
          console.log(response.data);
          this.products.push(response.data);
          this.newName = "";
          this.newDescription = "";
          this.newPrice = "";
          this.newImageUrl = "";
          this.errors = [];
        })
        .catch(error => {
          console.log(error.response.data);
          this.errors = error.response.data.errors;
        });
    },
    showProduct: function(product) {
      this.currentProduct = product;
      document.querySelector("dialog").showModal();
    },
    updateProduct: function(product){
      var params = {
        name: product.name,
        description: product.description,
        price: product.price,
        image_url: product.image_url,
      };

      axios
        .patch(`/api/products/${product.id}`, params)
        .then(response => {
          console.log("success!", response.data);
        })
        .catch(error => {
          console.log(error.response.data.errors);
        });
    },
    destroyProduct: function(product) {
      axios
        .delete(`/api/products/${product.id}`)
        .then(response => {
          console.log(response.data);
          var index = this.products.indexOf(product);
          this.products.splice(index, 1);
        })
        .catch(error => {
          console.log(error);
        });
    },
  },
};
</script>
