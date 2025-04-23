<script setup>
import { onMounted, ref, watch, reactive, provide, computed } from 'vue'
import axios from 'axios'

import Card from '../components/Card.vue'
import Header from '../components/Header.vue'
import CardList from '../components/CardList.vue'
import Drawer from '../components/Drawer.vue'
import debounce from 'lodash.debounce'

const items = ref([])
const cart = ref([])


const drawerOpen = ref(false)


const totalPrice = computed(
  () => cart.value.reduce((acc, item) => acc + item.price, 0)
)
 
const vatPrice = computed(
  () => Math.round((totalPrice.value * 5) / 100) 
)



const closeDrawer = () => {
  drawerOpen.value = false
}

const openDrawer = () => {
  drawerOpen.value = true
}

const addToCart = (item) => {
  cart.value.push(item)
  item.isAdded = true
}

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1)
  item.isAdded = false
}




provide('cart', {
  cart,
  closeDrawer,
  openDrawer,
  addToCart,
  removeFromCart
})




const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

// для закладок
const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://3345c2b695289b73.mokky.dev/favorites`)

    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id)

      if (!favorite) {
        return item
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (err) {
    console.log(err)
  }
}

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id,
      }

      item.isFavorite = true

      const { data } = await axios.post(`https://3345c2b695289b73.mokky.dev/favorites`, obj)

      item.favoriteId = data.id
    } else {
      item.isFavorite = false

      await axios.delete(`https://3345c2b695289b73.mokky.dev/favorites/${item.favoriteId}`)
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

//для сортировки
const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 300)


const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
      // searchQuery: filters.searchQuery,
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }

    const { data } = await axios.get(`https://3345c2b695289b73.mokky.dev/items`, {
      params
    })

    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }))
  } catch (err) {
    console.log(err)
  }
}

//вытаскиваем из бд все данные
onMounted(async () => {
  const localCart = localStorage.getItem('cart');
  cart.value = localCart ? JSON.parse(localCart) : [];


  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) =>({
      ...item,
      isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

//тоже для сортировки, отлавливает изменения
watch(filters, fetchItems)

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item, 
    isAdded: false
  }))
})

watch(cart, () =>{
  localStorage.setItem('cart', JSON.stringify(cart.value))
}, 
{ deep: true}
)

</script>

<template>
  <Drawer 
   v-if="drawerOpen" 
   :total-price="totalPrice"
   :vat-price="vatPrice" 
    />
  <div class="bg-white w-3/5 m-auto rounded-xl shadow-xl shadow-grey-200 mt-20">
    <Header
    :total-price="totalPrice"
     @open-drawer="openDrawer" />

    <div class="p-10">
      <div class="flex justify-between items-center mb-10">
        <h1 class="text-3xl font-bold">Все кроссовки</h1>
        <div class="flex items-center gap-4">
          <select
            @change="onChangeSelect"
            class="py-2 px-3 border border-gray-200 focus:border-gray-400 rounded-md outline-none"
          >
            <option value="name">По названию</option>
            <option value="price">По цене (дешевые)</option>
            <option value="-price">По цене (дорогие)</option>
          </select>
          <div class="relative">
            <input
              @input="onChangeSearchInput"
              type="text"
              class="border border-gray-200 rounded-md py-2 pl-10 pr-4 outline-none focus:border-gray-400"
              placeholder="Поиск..."
            />
            <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
              <img src="/search.svg" />
            </div>
          </div>
        </div>
      </div>

      <div class="grid grid-cols-4 gap-10" v-auto-animate>
        <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
      </div>
    </div>
  </div>
</template>
