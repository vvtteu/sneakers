<script setup>

import Header from '../components/Header.vue'
import axios from 'axios'

import { onMounted, ref } from 'vue'
import CardList from '../components/CardList.vue'


const favorites = ref([]);

onMounted( async () => {
     try {
        const {data} = await axios.get('https://3345c2b695289b73.mokky.dev/favorites?_relations=items'

        )

        favorites.value = data.map(obj => obj.item)
     } catch (err){
        console.log(err)
     }
});

</script>

<template>
 <div class="bg-white w-3/5 m-auto rounded-xl shadow-xl shadow-grey-200 mt-20">
    <Header />
         <div class="p-10">
            <h1 class="text-3xl font-bold">Мои закладки</h1>
                  <div class="grid grid-cols-4 gap-10" v-auto-animate>
                  <CardList :items="favorites" is-favorites/>
                  </div>
         </div>
    </div>
</template>