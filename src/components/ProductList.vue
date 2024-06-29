<template>
  <div class="row">
    <div class="col-sm-6" v-for="product in sortedLessons" :key="product.id">
      <div class="card mb-3 mt-3 rounded border border-success-subtle">
        <div class="card-header d-flex">
          <div>
            <h5>title: {{ product.title }}</h5>
          </div>
          <i :class="product.icon" class="fs-1 ms-auto"></i>
        </div>
        <div class="card-body">
          <p>price: {{ product.price }} AED</p>
          <p>Spaces: {{ product.spaces }}</p>
          <p>location: {{ product.location }}</p>
        </div>
        <div class="card-footer">
          <button
            class="btn btn-info rounded"
            v-on:click="addToCart(product)"
            :disabled="product.spaces === 0"
          >
            Add to cart
          </button>
          <button
            class="btn btn-danger rounded"
            v-on:click="removeProduct(product)"
          >
            Remove
          </button>
        </div>
      </div>
    </div>
  </div>

  <div class="container mt-4"></div>
  <br />
  <div class="row g-3">
    <div class="col-sm-8 border rounded-3">
      <div v-for="item in cart" class="border-bottom" :key="item.lesson.id">
        <div class="col-sm-12 mb-4 p-3">
          <div class="row">
            <div class="col-sm-4">
              <i :class="item.lesson.icon" class="display-1 ms-auto"></i>
            </div>
            <div class="col-sm-4">
              <p>Title: {{ item.lesson.title }}</p>
              <p>quantity: {{ item.amount }}</p>
              <button
                class="btn btn-danger rounded"
                v-on:click="removeProduct(item.lesson)"
              >
                Remove
              </button>
            </div>
            <div class="col-sm-4">
              <p>price: {{ item.lesson.price }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>



 <script>
export default {
  name: "ProductList",
  data() {
    return {
      showProduct: true,
      lessons: [],
      cart: JSON.parse(localStorage.getItem("cart")) || [],
      searchTerm: "",
      username: "",
      phonenumber: "",
      sortAttribute: "title",
      sortOrder: "asc",
      sortedLessons: [],
    };
  },
  created() {
    this.getLessons();
  },
  methods: {
    async getLessons() {
      try {
        const response = await fetch(
          "https://cst-3145-cw-2.vercel.app/collection/Products/"
        );
        this.lessons = await response.json();
        this.searchLessons();
      } catch (error) {
        console.error("Error fetching lessons:", error);
      }
    },
    searchLessons() {
      const searchTermLower = this.searchTerm.toLowerCase();
      this.sortedLessons = this.lessons
        .filter(
          (lesson) =>
            lesson.title.toLowerCase().includes(searchTermLower) ||
            lesson.location.toLowerCase().includes(searchTermLower)
        )
        .sort((a, b) => {
          let comparison = 0;
          if (typeof a[this.sortAttribute] === "string") {
            comparison = a[this.sortAttribute].localeCompare(
              b[this.sortAttribute]
            );
          } else {
            comparison = a[this.sortAttribute] - b[this.sortAttribute];
          }
          return this.sortOrder === "asc" ? comparison : -comparison;
        });
    },
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        lesson.spaces--;
        const cartItem = this.cart.find((item) => item.lesson.id === lesson.id);
        if (cartItem) {
          cartItem.amount++;
        } else {
          this.cart.push({ lesson, amount: 1 });
        }
      }
    },
    removeProduct(lesson) {
      const index = this.cart.findIndex((item) => item.lesson.id === lesson.id);
      if (index !== -1) {
        this.cart[index].amount--;
        lesson.spaces++;
        if (this.cart[index].amount === 0) {
          this.cart.splice(index, 1);
        }
      }
    },
    showCheckOut() {
      this.showProduct = !this.showProduct;
    },
    async checkout() {
      const order = {
        lessons: this.cart,
        username: this.username,
        phonenumber: this.phonenumber,
      };
      try {
        await fetch("https://cst-3145-cw-2.vercel.app/collection/orders/", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(order),
        });
        this.cart = [];
        alert("Order successful");
      } catch (error) {
        console.error("Error during checkout:", error);
      }
    },
  },
  computed: {
    cartSize() {
      return this.cart.reduce((sum, item) => sum + item.amount, 0);
    },
    validateUserCredentials() {
      return (
        /^[a-zA-Z]+$/.test(this.username) && /^\d+$/.test(this.phonenumber)
      );
    },
  },
};
</script>