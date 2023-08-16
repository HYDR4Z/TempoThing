<script setup>
  import { RouterLink, RouterView } from 'vue-router';
  import Login from './components/Login.vue';
  import { onMounted, ref, watch } from 'vue';
  import axios from 'axios';

  const code = new URLSearchParams(window.location.search).get('code');

  const accessToken = ref('');
  const refreshToken = ref('');
  const expiresIn = ref('');

  onMounted(() => {
    if (!code) return;
    axios.post(`${import.meta.env.VITE_AUTH_ENDPOINT}/login`, {
      code
    }).then(res => {
      accessToken.value = res.data.accessToken;
      refreshToken.value = res.data.refreshToken;
      expiresIn.value = res.data.expiresIn;
    })
  });

  // watch([accessToken, expiresIn], async () => {
  //   if (!accessToken.value || !expiresIn.value) return;
  //   console.log(refreshToken.value, expiresIn.value)
  //   const refreshInterval = setInterval(() => {
  //     axios.post(`${import.meta.env.VITE_AUTH_ENDPOINT}/refresh`, {
  //       code
  //     }).then(res => {
  //       accessToken.value = res.data.accessToken;
  //       expiresIn.value = res.data.expiresIn;
  //     })
  //   }, (expiresIn - 60) * 1000);
  //   return () => clearInterval(refreshInterval);
  // }, { immediate: true });
</script>

<template>
  <header>
    <h1>TempoThing</h1>
    <div class="nav-wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
      </nav>
      <Login v-if="!accessToken" />
    </div>
  </header>
  <main>
    <RouterView />
  </main>
</template>
