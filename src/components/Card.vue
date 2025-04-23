<script setup>
import { ref } from 'vue';



const props = defineProps({
  id: Number,
  title: String,
  img: String,
  price: Number,
  isAdded: Boolean,
  isFavorite: Boolean,
   onClickAdd: Function,
   onClickFavorite: Function
});

// const isLiked = ref(false);

// const onClick = () => {
//   isLiked.value = !isLiked.value;
// };
const visibleFavoriteButton = Boolean(props.onClickAdd)


</script>

<template>
  <div
    class="relative flex flex-col w-full border border-slate-100 rounded-xl p-8 cursor-pointer transition hover:shadow-xl hover:transform hover:-translate-y-2"
  >
    <div @click="onClickFavorite" class="absolute top-8 left-8">
      <img 
      v-if="visibleFavoriteButton"
      :src="!isFavorite ? '/like-1.svg' : '/like-2.svg'" alt="Favorite" />
    </div>
    <img :src="img" class="w-full" alt="Sneaker" />
    <p>{{ title }}</p>
    <div class="flex justify-between mt-5">
      <div class="flex flex-col gap-2">
        <span class="text-slate-200">Цена:</span>
        <span class="font-bold">{{ price }} руб.</span>
      </div>
      <img 
      v-if="onClickFavorite"
      @click="onClickAdd" :src="!isAdded ? '/plus.svg' : '/checked.svg'  " alt="Plus" />
    </div>
  </div>
</template>
