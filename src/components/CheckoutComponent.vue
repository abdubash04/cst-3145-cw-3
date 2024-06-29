<template>
  <div class="container mt-4">
    <h1>Checkout Page</h1>
    <div v-for="item in cartItems" :key="item._id" class="card mb-3">
      <div class="card-body">
        <p><strong>{{ item.title }}</strong> (Quantity: {{ item.quantity }})</p>
        <p>Location: {{ item.location }}</p>
        <p>Price: ${{ item.price }}</p>
        <button @click="removeFromCart(item)" class="btn btn-danger">Remove</button>
      </div>
    </div>
    <div class="form-group">
      <label>Name: </label>
      <input v-model="name" type="text" pattern="^[a-zA-Z\s]+$" required class="form-control">
    </div>
    <div class="form-group">
      <label>Phone number: </label>
      <input v-model="phone" type="text" pattern="^\d+$" required class="form-control">
    </div>
    <button :disabled="!validForm" @click="checkout" class="btn btn-primary">Checkout</button>
    <p v-if="!validForm" style="color: red;">Please fill out all fields correctly.</p>
    <p v-if="checkoutMessage" style="color: green;">{{ checkoutMessage }}</p>
  </div>
</template>

<script>
export default {
  name: "CheckoutComponent",
  props: {
    cart: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      name: "",
      phone: "",
      checkoutMessage: "",
    };
  },
  computed: {
    cartItems() {
      return this.cart;
    },
    validForm() {
      return this.name.trim() !== "" && this.phone.match(/^\d+$/);
    }
  },
  methods: {
    removeFromCart(item) {
      this.$emit('remove-from-cart', item);
    },
    async checkout() {
      if (!this.validForm) {
        return;
      }
      try {
        const order = {
          lessons: this.cartItems,
          username: this.name,
          phonenumber: this.phone,
        };
        const response = await fetch('https://cst-3145-cw-2.vercel.app/collection/orders/', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(order),
        });
        if (!response.ok) {
          throw new Error(`Error creating order: ${response.statusText}`);
        }
        this.checkoutMessage = `Order created successfully for ${this.name} (phone: ${this.phone})!`;
        this.$emit('clear-cart');
        setTimeout(() => {
          this.$router.push('/');
        }, 2000); // Redirect back to home after 2 seconds
      } catch (error) {
        console.error('Error:', error.message);
        alert(`Failed to create order: ${error.message}`);
      }
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 600px;
}
.card {
  margin-bottom: 1rem;
}
.btn-primary {
  margin-top: 1rem;
}
</style>
