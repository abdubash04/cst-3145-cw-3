
<template>
<div class="d-flex align-items-end gap-3 flex-wrap">
                    <div>
                        <input type="text" class="form-control" v-model="searchTerm"
                            placeholder="search for a lesson..." v-on:input="searchLessons">
                    </div>

                    <div>
                        <label for="sortAttribute">Sort By:</label>
                        <select v-model="sortAttribute" class="form-select" v-on:change="searchLessons">
                            <option value="title">Title</option>
                            <option value="location">Location</option>
                            <option value="price">Price</option>
                            <option value="spaces">Spaces</option>
                        </select>
                    </div>
                    <div>
                        <label for="sortOrder">Sort Order:</label>
                        <select v-model="sortOrder" class="form-select" v-on:change="searchLessons">
                            <option value="asc">Ascending</option>
                            <option value="desc">Descending</option>
                        </select>
                    </div>
                </div>
  <div class="row">
    <div class="col-sm-6" v-for="product in sortedLessons" :key="product.id">
      <div class="card mb-3 mt-3 rounded border border-success-subtle">
        <div class="card-header d-flex">
          <div>
            <h5>Title: {{ product.title }}</h5>
          </div>
          <i :class="product.icon" class="fs-1 ms-auto"></i>
        </div>
        <div class="card-body">
          <i :class="lessons.icon"></i>
          <p>Price: {{ product.price }} AED</p>
          <p>Spaces: {{ product.spaces }}</p>
          <p>Location: {{ product.location }}</p>
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
      sortAttribute: "title",
      sortOrder: "asc",
      sortedLessons: []
    };
  },
  created() {
    this.getLessons();
  },
  methods: {
    async getLessons() {
      try {
        const response = await fetch("https://cst-3145-cw-2.vercel.app/collection/Products/");
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
        const cartItem = this.cart.find((item) => item.lesson === lesson);
        if (cartItem) {
          cartItem.amount++;
        } else {
          this.cart.push({ lesson, amount: 1 });
        }
        localStorage.setItem('cart', JSON.stringify(this.cart));
        this.$emit('update-cart', lesson);
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
        localStorage.setItem('cart', JSON.stringify(this.cart));
        this.$emit('remove-from-cart', lesson);
      }
    },
    showCheckOut() {
      this.showProduct = !this.showProduct;
    },
  },
  computed: {
    cartSize() {
      return this.cart.reduce((sum, item) => sum + item.amount, 0);
    },
  },
};
</script>
