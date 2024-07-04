<template>
  <div class="container mt-4">
    <h1>Checkout Page</h1>
    <div v-for="item in cartItems" :key="item.lesson.id" class="card mb-3">
      <div class="card-body">
        <p><strong>{{ item.lesson.title }}</strong> (Quantity: {{ item.amount }})</p>
        <p>Location: {{ item.lesson.location }}</p>
        <p>Price: ${{ item.lesson.price }}</p>
        <button @click="removeFromCart(item.lesson)" class="btn btn-danger">Remove</button>
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
      console.log('Cart items:', this.cart); // Debugging line
      return this.cart;
    },
    validForm() {
      return this.name.trim() !== "" && this.phone.match(/^\d+$/);
    }
  },
  methods: {
    removeFromCart(lesson) {
      console.log('Removing lesson:', lesson); // Debugging line
      this.$emit('remove-from-cart', lesson);
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
        console.log('Placing order:', order); // Debugging line
        const response = await fetch('https://cst-3145-cw-2.vercel.app/collection/orders/', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(order),
        });
        if (!response.ok) {
          const errorText = await response.text();
          throw new Error(`Error creating order: ${response.statusText} - ${errorText}`);
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
