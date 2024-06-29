<template>
  <div id="app" class="container">
    <header>
      <h2>A1 Lessons</h2>
    </header>
    <main>
      <ProductList @addProducts="addToCart" :serverUrl="serverUrl" v-if="activeComponent === 'ProductList'" />
      <CheckoutComponent :cart="cart" v-if="activeComponent === 'CheckoutComponent'" @remove-from-cart="removeFromCart" @clear-cart="clearCart"/>
      <button v-on:click="changeComponent" class="btn btn-primary m-3">Proceed to Checkout ({{ cartItemCount }})</button>
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
      serverUrl: "https://cst-3145-cw-2.vercel.app/collection/orders/",
      cart: JSON.parse(localStorage.getItem('cart')) || [],
      activeComponent: 'ProductList'
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
      if (lesson.spaces > 0) {
        const itemInCart = this.cart.find(item => item._id === lesson._id);
        if (itemInCart) {
          itemInCart.quantity++;
        } else {
          this.cart.push({ ...lesson, quantity: 1 });
        }
        lesson.spaces--;
        localStorage.setItem("cart", JSON.stringify(this.cart));
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
  },
};
</script>
