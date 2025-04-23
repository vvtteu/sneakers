<script setup>
import Card from './Card.vue'
// import {inject} from 'vue'

defineProps({
    items: Array,
    isFavorites: Boolean
})

const emit = defineEmits(['addToFavorite' , 'addToCart']);


</script>


<template>

        <Card
          v-for="item in items"
          :key="item.id"
          :id="item.id"
          :title="item.title"
          :price="item.price"
          :img="item.imageUrl"
          :onClickFavorite="isFavorites ? null : () =>  emit('addToFavorite', item)"
          :onClickAdd="isFavorites ? null : () =>  emit('addToCart', item)"
          :isFavorite="item.isFavorite"
          :isAdded="item.isAdded"
        />

</template>