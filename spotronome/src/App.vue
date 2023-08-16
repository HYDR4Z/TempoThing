<script setup>
  import { RouterLink, RouterView } from 'vue-router'
  import Login from './components/Login.vue'
  import { watchEffect } from 'vue';
  import axios from 'axios';

  const code = new URLSearchParams(window.location.search).get('code');
  watchEffect(() => {
    if (!code) return;
    axios.post(`${import.meta.env.VITE_AUTH_ENDPOINT}/login`, {
      code
    }).then(res => {
      console.log(res.data)
    }).catch(() => window.location = '/');
  });
</script>

<template>
  <header>
    <h1>Tempothing</h1>
    <div class="nav-wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
      </nav>
      <Login v-if="!code" />
    </div>
  </header>

  <main>
    <RouterView />
  </main>
</template>
