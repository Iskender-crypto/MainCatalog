<script setup>
import {computed, onMounted, ref, watch} from "vue";
import axios from "axios";

const response = ref(false);
const timeOut = ref();
const searchField = ref('Batman');
const movies = ref([]);
const currentPage = ref(1);
const totalResults = ref(0);
const message = ref('');
const hasNextPage = computed(() => {
  return currentPage.value * 10 < totalResults.value;
});
const computedResponse = computed(() => response.value);

const nextPage = () => {
  if (hasNextPage.value) {
    currentPage.value++;
    getMovies();
  }
};

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
    getMovies();
  }
};

const getMovies = async () => {
  try {
    const {data} = await axios.get(`https://www.omdbapi.com/?apikey=8523cbb8&s=${searchField.value}&page=${currentPage.value}`);
    if (data.Response === "True") {
      movies.value = data.Search || [];
      totalResults.value = parseInt(data.totalResults) || 0;
      response.value = true;
    } else {
      movies.value = [];
      totalResults.value = 0;
      response.value = false;
    }
  } catch (error) {
    console.error("Ошибка при получении данных с сервера", error);
    movies.value = [];
    totalResults.value = 0;
    response.value = false;
  }
};

onMounted(async () => {
  await getMovies();
});

watch(
    () => searchField.value,
    async () => {
      clearTimeout(timeOut.value);
      timeOut.value = setTimeout(async () => {
        currentPage.value = 1;
        await getMovies();
      }, 1300);
    }
);

const pageCount = computed(() => Math.ceil(totalResults.value / 10));
</script>
<template>
  <header class="header">
    <div class="container">
      <div class="navbar">
        <h2 class="navbar-brand">Movie Catalog</h2>
        <input v-model="searchField" type="search" placeholder="search" class="navbar__search search">
        <div class="navbar__profile">
          <img src="https://cdn-icons-png.freepik.com/512/1077/1077114.png" alt="">
          <select name="" id="" class="select">
            <option value="1" selected>Alexander Borisenko</option>
            <option value="2">Iskender Azamov</option>
          </select>
        </div>
      </div>
    </div>
  </header>
  <main class="content">
    <div class="container">
      <h1 class="content__title">
        You searched for: {{ searchField }}, {{ totalResults }} results found
      </h1>
      <div class="content__flex">
        <div class="content__col" v-for="item in movies" :key="item.imdbID">
          <div class="card">
            <div class="card__img-placeholder" v-if="item.Poster === 'N/A'">
              <img src="https://pixsector.com/cache/517d8be6/av5c8336583e291842624.png" alt="">
            </div>
            <img class="card__img" :src="item.Poster" alt="" v-else>
            <div class="card__content">
              <div class="card__content-flex">
                <b>Name: </b>
                <p>{{ item.Title }}</p>
              </div>
              <div class="card__content-flex">
                <b>Year: </b>
                <p>{{ item.Year }}</p>
              </div>
              <div class="card__content-flex">
                <b>imdbID: </b>
                <p>{{ item.imdbID }}</p>
              </div>
              <div class="card__content-flex">
                <b>Type: </b>
                <p>{{ item.Type }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="content__pagination pagination">
        <button @click="prevPage" :disabled="currentPage === 1">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-left-short" viewBox="0 0 16 16">
            <path fill-rule="evenodd" d="M12 8a.5.5 0 0 1-.5.5H5.707l2.147 2.146a.5.5 0 0 1-.708.708l-3-3a.5.5 0 0 1 0-.708l3-3a.5.5 0 1 1 .708.708L5.707 7.5H11.5a.5.5 0 0 1 .5.5"/>
          </svg>
        </button>
        <div class="pagination__numbers">
          <button
              v-for="page in pageCount"
              :key="page"
              @click="currentPage = page; getMovies();"
              :class="{ active: currentPage === page }"
          >{{ page }}</button>
        </div>
        <button @click="nextPage" :disabled="!hasNextPage">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-right-short" viewBox="0 0 16 16">
            <path fill-rule="evenodd" d="M4 8a.5.5 0 0 1 .5-.5h5.793L8.146 5.354a.5.5 0 1 1 .708-.708l3 3a.5.5 0 0 1 0 .708l-3 3a.5.5 0 0 1-.708-.708L10.293 8.5H4.5A.5.5 0 0 1 4 8"/>
          </svg>
        </button>
      </div>
    </div>
  </main>
  <div class="message" v-if="!response">
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-exclamation-diamond-fill" viewBox="0 0 16 16">
      <path d="M9.05.435c-.58-.58-1.52-.58-2.1 0L.436 6.95c-.58.58-.58 1.519 0 2.098l6.516 6.516c.58.58 1.519.58 2.098 0l6.516-6.516c.58-.58.58-1.519 0-2.098zM8 4c.535 0 .954.462.9.995l-.35 3.507a.552.552 0 0 1-1.1 0L7.1 4.995A.905.905 0 0 1 8 4m.002 6a1 1 0 1 1 0 2 1 1 0 0 1 0-2"/>
    </svg>
    нет результатов
  </div>
</template>
<style scoped>

</style>
