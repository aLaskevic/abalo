<script setup>
import { ref } from "vue";

const inputValue = ref("");
const inputLength = ref(0);
const data = ref([]);
const INPUT_LIMIT = ref(1);
const PAGE_SIZE = ref(5);
const CURR_PAGE = ref(0);

async function getData(pageSize, currPage) {
  const res = await fetch(
    `http://127.0.0.1:8000/api/articles?search=${inputValue.value}&offset=${
      pageSize * currPage + 1
    }&limit=${pageSize}`
  );
  data.value = await res.json();
}

function handleClick(event, index) {
  CURR_PAGE.value = index;
  getData(PAGE_SIZE.value, index);
  window.scrollTo({ top: 0, behavior: "smooth" });
  document.querySelectorAll(".pages > li").forEach((entry) => {
    console.log("TTTT");
    console.log(entry.classList.contains("active"));
    if (entry.classList.contains("active")) entry.classList.remove("active");
  });

  console.log("EVENTTARGET:", event.target.classList);
  event.target.classList.add("active");
}

function imageExists(image_url) {
  var http = new XMLHttpRequest();

  http.open("HEAD", image_url, false);
  http.send();

  return http.status != 404;
}

function getPageCount() {
  console.log(data.value.data.length);
  if (data.value.data.length > 0)
    return parseInt(Math.ceil(this.data.size / this.PAGE_SIZE));
  else return 0;
}

function updateLength() {
  console.log(inputValue.length);
  inputLength.value = inputValue.value.length;
  if (inputLength.value >= INPUT_LIMIT.value) {
    getData(PAGE_SIZE.value, CURR_PAGE.value);
  }
}
</script>

<style scoped src="./ArticleSearch.css"></style>
<template>
  <div
    class="logo"
    :class="
      inputValue.length >= INPUT_LIMIT && data.data?.length > 0
        ? 'logo-active'
        : 'logo-inactive'
    "
  >
    <img
      src="abalo.png"
      alt=""
      class="logo"
      :class="
        inputValue.length >= INPUT_LIMIT && data.data?.length > 0
          ? 'image-active'
          : 'image-inactive'
      "
    />
  </div>
  <div
    class="searchbar"
    :class="
      inputValue.length >= INPUT_LIMIT && data.data?.length > 0
        ? 'searchbar-active'
        : 'searchbar-inactive'
    "
  >
    <input
      v-model="inputValue"
      @input="updateLength"
      placeholder="Search your article!"
      type="text"
    />
    <button>Search</button>
  </div>
  <ul v-if="inputLength >= INPUT_LIMIT">
    <li class="article-entry" v-for="article in data.data" :key="article.id">
      <div class="buy-button-section">
        <button class="article-buy-button">Add to cart</button>
        <button class="article-buy-button">Buy</button>
      </div>
      <img
        v-if="
          imageExists(
            'http://127.0.0.1:8000/articleimages/' + article.id + '.jpg'
          )
        "
        :src="'http://127.0.0.1:8000/articleimages/' + article.id + '.jpg'"
      />
      <div v-else class="image-not-found">
        <div>Image not found</div>
      </div>
      <div class="article-entry-informations">
        <div class="article-entry-informations-name">
          {{ article.name }}
        </div>
        <div class="article-entry-informations-description">
          {{ article.description }}
        </div>
        <div class="article-entry-informations-price">
          {{ article.price }} <span>€</span>
        </div>
      </div>
    </li>
  </ul>
  <ul class="pages" v-if="data.data && data.size > PAGE_SIZE">
    <li
      v-for="index in getPageCount()"
      :key="index"
      @click="(event) => handleClick(event, index - 1)"
    >
      {{ index }}
    </li>
  </ul>
</template>
