<script setup>
import { ref, computed, onMounted } from 'vue'

const query = ref('')
const my_anime = ref([])
const search_results = ref([])

const my_anime_asc = computed(() => {
  return my_anime.value.sort((a, b) => {
    return a.title.localeCompare(b.title)
  })
})

const searchAnime = () => {
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`
  fetch(url)
    .then(res => res.json())
    .then(res => {
      search_results.value = res.data
    })
}

const handleInput = evt => {
  if (!evt.target.value) {
    search_results.value = []
  }
}

const addAnime = anime => {
  search_results.value = []
  query.value = ''

  my_anime.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    total_episodes: anime.episodes,
    watched_episodes: 0
  })

  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}

const removeAnime = anime => {
  my_anime.value = my_anime.value.filter(a => a !== anime)
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
}


const increaseWatch = anime => {
  anime.watched_episodes++
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))

}

const decreaseWatch = anime => {
  anime.watched_episodes--
  localStorage.setItem('my_anime', JSON.stringify(my_anime.value))

}

onMounted(() => {
  my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || []
})

</script>

<template>
  <main>
    <h1>Trackanime 〽️</h1>
    <form @submit.prevent="searchAnime">
      <input type="text" placeholder="Search an anime..." v-model="query" @input="handleInput">

      <button type="submit" class="button">Search</button>
    </form>

    <div class="results" v-if="search_results.length > 0">
      <div class="result" v-for="anime in search_results">
        <img :src="anime.images.jpg.image_url" />
        <div class="details">
          <h3>{{ anime.title }}</h3>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}...
          </p>
          <span class="flex-1"></span>
          <button class="button" @click="addAnime(anime)">Add to my list</button>
        </div>
      </div>
    </div>

    <div class="myanime" v-if="my_anime.length > 0">
      <h2>My anime</h2>

      <div class="anime" :title="anime.title" v-for="anime in my_anime_asc">
        <img :src="anime.image" alt="">
        <h3 v-if="anime.title.length > 20">{{ anime.title.slice(0, 20) }}...</h3>
        <h3 v-else>{{ anime.title }}</h3>
        <div class="flex-1"></div>
        <span class="episodes">
          {{ anime.watched_episodes}} / {{ anime.total_episodes }}
        </span>

        <button v-if="anime.total_episodes !== anime.watched_episodes" @click="increaseWatch(anime)" class="button">
          +
        </button>

        <button v-if=" anime.watched_episodes > 0" @click="decreaseWatch(anime)" class="button">
          -
        </button>

        <div class="actions">
          <button class="button" @click="removeAnime(anime)">🗑</button>
        </div>
      </div>
    </div>

  </main>
</template>

