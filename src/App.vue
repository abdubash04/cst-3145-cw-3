<template>
  <div id="app" class="container">
    <header>
      <h2>A1 Lessons</h2>
    </header>
    <main>
      <ProductList @update-cart="addToCart" @remove-from-cart="removeFromCart" v-if="activeComponent === 'ProductList'" />
      <CheckoutComponent :cart="cart" v-if="activeComponent === 'CheckoutComponent'" @remove-from-cart="removeFromCart" @clear-cart="clearCart"/>
      <button v-if="activeComponent === 'ProductList'" v-on:click="changeComponent" class="btn btn-primary m-3">Proceed to Checkout ({{ cartItemCount }})</button>
    </main>
  </div>
</template>

<script>
import CheckoutComponent from "./components/CheckoutComponent.vue";
import ProductList from "./components/ProductList.vue";

export default {
  name: "App",
  components: {
    ProductList,
    CheckoutComponent
  },
  data() {
    return {
      cart: JSON.parse(localStorage.getItem('cart')) || [],
      activeComponent: 'ProductList',
      items: []
    };
  },
  computed: {
    cartItemCount() {
      return this.cart.reduce((total, item) => total + item.quantity, 0);
    }
  },
  methods: {
    changeComponent() {
      this.activeComponent = this.activeComponent === 'ProductList' ? 'CheckoutComponent' : 'ProductList';
    },
    addToCart(lesson) {
      console.log('addToCart called');
      if (lesson.spaces > 0) {
        const itemInCart = this.cart.find(item => item._id === lesson._id);
        if (itemInCart) {
          itemInCart.quantity++;
          console.log(`Increased quantity for item ${itemInCart._id}`);
        } else {
          this.cart.push({ ...lesson, quantity: 1 });
          console.log(`Added item ${lesson._id} to cart`);
        }
        lesson.spaces--;
        console.log(`Remaining spaces for item ${lesson._id}: ${lesson.spaces}`);
        localStorage.setItem("cart", JSON.stringify(this.cart));
        console.log('Cart updated in local storage');
      } else {
        console.log('No spaces available');
      }
    },
    removeFromCart(item) {
      const itemIndex = this.cart.findIndex(cartItem => cartItem._id === item._id);
      if (itemIndex !== -1) {
        if (this.cart[itemIndex].quantity > 1) {
          this.cart[itemIndex].quantity--;
        } else {
          this.cart.splice(itemIndex, 1);
        }
        localStorage.setItem('cart', JSON.stringify(this.cart));
      }
    },
    clearCart() {
      this.cart = [];
      localStorage.setItem('cart', JSON.stringify(this.cart));
    }
  }
};
</script>
