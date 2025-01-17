<script setup>
import { ref, watch, onMounted } from "vue";
import axios from "axios";
import { inject } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import { Autoplay, Pagination, Navigation } from "swiper/modules";
import "swiper/css";
import ItemList from "../components/ItemList.vue";

const modules = [Autoplay, Pagination, Navigation];

const carouselFhotos = [
  "MARGINIS_CAROUSEL_SKATE_00.jpg",
  "MARGINIS_CAROUSEL_SKATE_01.jpg",
  "MARGINIS_CAROUSEL_SKARE_02.ARW",
];

const { cart, addToCart, removeFromCart } = inject("cart");

const items = ref([]);
const searchQuerry = ref(null);

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(
      `https://e9b03337b16fcf1d.mokky.dev/favorites`
    );
    items.value = items.value.map((item) => {
      const favorite = favorites.find(
        (favorite) => favorite.parentId === item.id
      );
      if (!favorite) {
        return item;
      }
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id,
      };
    });
  } catch (err) {
    console.log(err);
  }
};
const fetchItems = async () => {
  try {
    const params = {
      title: searchQuerry.value,
    };
    if (searchQuerry.value) {
      params.title = `*${searchQuerry.value}*`;
    } else params.title = null;
    const { data } = await axios.get(
      `https://e9b03337b16fcf1d.mokky.dev/items`,
      { params }
    );
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      isAdded: false,
      favoriteId: null,
    }));
  } catch (err) {
    console.log(err);
  }
};

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        parentId: item.id,
        item,
      };
      item.isFavorite = true;

      const { data } = await axios.post(
        `https://e9b03337b16fcf1d.mokky.dev/favorites`,
        obj
      );

      item.favoriteId = data.id;
    } else {
      item.isFavorite = false;

      await axios.delete(
        `https://e9b03337b16fcf1d.mokky.dev/favorites/${item.favoriteId}`
      );

      item.favoriteId = null;
    }
  } catch (err) {
    console.log(err);
  }
};

const onCangeSearchInput = (event) => {
  searchQuerry.value = event.target.value;
};

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item);
  } else {
    removeFromCart(item);
  }
};

onMounted(async () => {
  const localCart = localStorage.getItem("cart");
  cart.value = localCart ? JSON.parse(localCart) : [];

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id),
  }));
  await fetchItems();
  await fetchFavorites();
});

watch(searchQuerry, fetchItems);

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false,
  }));
});
</script>
<template>
  <swiper
    :centeredSlides="true"
    :autoplay="{
      delay: 5000,
      disableOnInteraction: false,
    }"
    :pagination="{
      clickable: true,
    }"
    :navigation="true"
    :modules="modules"
    class="swiper"
  >
    <swiper-slide v-for="(photo, index) in carouselFhotos" :key="index">
      <img :src="`../../../public/image/${photo}`" />
    </swiper-slide>
  </swiper>
  <div class="text-center text-red-400">
    <h1>SS autumn 2024</h1>
  </div>
  <div class="flex justify-end items-center mr-2 mb-2 mt-2">
    <div class="flex items-end gap-4">
      <div class="relative">
        <img
          src="../../public/image/search.svg"
          class="absolute left-3 top-3"
          alt="Search"
          width="20"
          height="20"
        />
        <input
          @input="onCangeSearchInput"
          class="border rounded-md py-2 pl-10 pr-4 outline-none focus:border-gray-400"
          placeholder="Search"
        />
      </div>
    </div>
  </div>
  <div class="mb-8">
    <ItemList
      :items="items"
      @add-to-favorite="addToFavorite"
      @add-to-cart="onClickAddPlus"
    />
  </div>
</template>
<style scoped>
.swiper {
  box-shadow: 0 25px 50px -12px rgb(0 0 0 / 0.25);
  width: 90%;
  height: 80vh;
  border: 10px solid rgb(198, 4, 4);
}
.swiper-slide img {
  width: 100%;
  height: 80vh;
}
</style>
