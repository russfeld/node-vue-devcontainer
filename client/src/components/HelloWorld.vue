<script setup>
defineProps({
  msg: {
    type: String,
    required: true
  }
})

import { onMounted, ref } from 'vue'
import axios from 'axios'

const data = ref("")

onMounted(() => {
  axios.get('/api/')
    .then(res => data.value = res.data)
    .catch(err => console.error(err))
})
</script>

<template>
  <div class="greetings">
    <h1 class="green">{{ msg }}</h1>
    <h3>
      You’ve successfully created a project with
      <a href="https://vitejs.dev/" target="_blank" rel="noopener">Vite</a> +
      <a href="https://vuejs.org/" target="_blank" rel="noopener">Vue 3</a>.
    </h3>
    <hr>
    <h3>Data from backend:</h3>
    <pre>{{ data }}</pre>
  </div>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
