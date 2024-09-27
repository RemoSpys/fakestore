<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const products = ref([]);
const cartItems = ref([]);
const isCartOpen = ref(false);

const fetchAllProducts = async () => {
  try {
    const response = await axios.get('https://fakestoreapi.com/products');
    products.value = response.data;
  } catch (error) {
    console.error('Error fetching products:', error);
  }
};

const addToCart = (product) => {
  const existingItem = cartItems.value.find(item => item.id === product.id);
  if (existingItem) {
    existingItem.quantity++;
  } else {
    cartItems.value.push({ ...product, quantity: 1 });
  }
};

const removeItem = (itemId) => {
  cartItems.value = cartItems.value.filter(item => item.id !== itemId);
};

const calculateTotal = () => {
  return cartItems.value.reduce((acc, item) => acc + (item.price * item.quantity), 0).toFixed(2);
};

const closeCart = (event) => {
  if (!event.target.closest('.cart')) {
    isCartOpen.value = false;
  }
};

const zoomImage = (event) => {
  const imgContainer = event.currentTarget;
  const rect = imgContainer.getBoundingClientRect();
  const x = ((event.clientX - rect.left) / rect.width) * 100;
  const y = ((event.clientY - rect.top) / rect.height) * 100;
  imgContainer.style.backgroundPosition = `${x}% ${y}%`;
  imgContainer.style.backgroundSize = '200%'; // Suurendame pildi suurust hoveri ajal
};

const resetZoom = (event) => {
  const imgContainer = event.currentTarget;
  imgContainer.style.backgroundSize = 'contain'; // Algne suurus, et pilt mahuks konteinerisse
  imgContainer.style.backgroundPosition = 'center'; // Algne positsioon
};

onMounted(async () => {
  await fetchAllProducts();
});
</script>

<template>
  <div class="min-h-screen flex flex-col bg-gray-100">
    <header class="bg-gray-900 text-white p-4">
      <div class="container mx-auto flex justify-between items-center">
        <h1 class="text-3xl font-bold">VõltsPood</h1>
        <div class="relative flex items-center" @click="isCartOpen = true">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 cursor-pointer" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 7H19m0 0l1.6 4H7m0 0L6 17h12.4m-8 2a1 1 0 100 2 1 1 0 000-2m6 0a1 1 0 100 2 1 1 0 000-2"/>
          </svg>
          <span class="absolute top-0 right-0 bg-red-500 text-white rounded-full text-xs w-5 h-5 flex items-center justify-center">
            {{ cartItems.length }}
          </span>
        </div>
      </div>
    </header>

    <main class="container mx-auto flex-1 p-8">
  <h2 class="text-4xl font-bold text-center mb-8">Tere tulemast meie võltspoodi</h2>
  <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-8">
    <article 
      v-for="product in products" 
      :key="product.id"
      class="bg-white shadow-lg rounded-lg overflow-hidden transition-transform transform hover:scale-105 flex flex-col"
      style="height: 500px;"
    >
      <div 
        class="w-full h-64 bg-white flex items-center justify-center overflow-hidden group"
        @mousemove="zoomImage"
        @mouseleave="resetZoom"
        :style="{ backgroundImage: `url(${product.image})`, backgroundSize: 'contain', backgroundPosition: 'center' }"
      >
        <!-- Siin pole vaja sisu, kasutame ainult tausta -->
      </div>
      <div class="p-4 flex-1 flex flex-col justify-between">
        <div>
          <h4 class="text-lg font-semibold mb-2">{{ product.title }}</h4>
          <p class="text-gray-600">{{ product.description.substring(0, 80) }}...</p>
        </div>
        <div class="mt-4">
          <div class="flex justify-between items-center mb-4">
            <span class="text-xl font-bold text-green-600">${{ product.price }}</span>
          </div>
          <button 
            class="bg-black text-white w-full py-2 rounded hover:bg-gray-800"
            @click="addToCart(product)"
          >
            Lisa ostukorvi
          </button>
        </div>
      </div>
    </article>
  </div>
</main>


    <div 
      v-if="isCartOpen" 
      @click="closeCart"
      class="fixed inset-0 bg-black bg-opacity-50 z-40"
    ></div>

    <div 
      class="cart fixed top-0 right-0 w-80 h-full bg-white shadow-lg transform transition-transform duration-300 z-50"
      :class="{ 'translate-x-0': isCartOpen, 'translate-x-full': !isCartOpen }"
    >
      <div class="p-4 border-b flex justify-between items-center">
        <h2 class="text-xl font-bold">Ostukorv</h2>
        <button @click="isCartOpen = false" class="text-gray-600 hover:text-gray-900">&times;</button>
      </div>
      <div class="p-4 flex-1 overflow-y-auto">
        <div v-if="cartItems.length > 0" class="space-y-4">
          <div v-for="item in cartItems" :key="item.id" class="flex justify-between items-center border-b pb-2">
            <div>
              <h4 class="font-semibold">{{ item.title }}</h4>
              <p class="text-sm text-gray-500">Kogus: {{ item.quantity }}</p>
              <p class="text-sm text-gray-500">Hind: ${{ item.price }}</p>
            </div>
            <button @click="removeItem(item.id)" class="bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600">
              Eemalda
            </button>
          </div>
        </div>
        <p v-else class="text-gray-500 text-center">Teie ostukorv on tühi.</p>
      </div>
      <div class="p-4 border-t flex justify-between items-center">
        <span class="font-semibold">Kogusumma: ${{ calculateTotal() }}</span>
        <button class="bg-black text-white px-4 py-2 rounded hover:bg-gray-800">Maksma</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    zoomImage(event) {
      const imgContainer = event.currentTarget;
      const rect = imgContainer.getBoundingClientRect();
      const x = ((event.clientX - rect.left) / rect.width) * 100;
      const y = ((event.clientY - rect.top) / rect.height) * 100;
      imgContainer.style.backgroundPosition = `${x}% ${y}%`;
      imgContainer.style.backgroundSize = '200%'; // Suurendame pildi suurust hoveri ajal
    },
    resetZoom(event) {
      const imgContainer = event.currentTarget;
      imgContainer.style.backgroundSize = 'contain'; // Taastame algse suuruse
      imgContainer.style.backgroundPosition = 'center'; // Taastame algse positsiooni
    }
  }
};
</script>
