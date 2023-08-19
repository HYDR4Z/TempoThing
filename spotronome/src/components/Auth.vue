<script setup>
  import { onMounted, ref, watch } from 'vue';
  import axios from 'axios';
  import SpotifyWebApi from 'spotify-web-api-node';

  const emit = defineEmits(['onAuthChanged']);

  const spotifyApi = new SpotifyWebApi({
    clientId: import.meta.env.VITE_CLIENT_ID
  });

  const AUTH_URL = "https://accounts.spotify.com/authorize?" + 
  new URLSearchParams({
    response_type: 'code',
    client_id: import.meta.env.VITE_CLIENT_ID,
    scope: 'user-read-email user-library-read user-read-playback-state user-modify-playback-state playlist-read-private user-read-private user-read-email',
    redirect_uri: import.meta.env.VITE_REDIRECT_URL
  }).toString();

  const code = new URLSearchParams(window.location.search).get('code');

  const accessToken = ref('');
  const refreshToken = ref('');
  const expiresIn = ref(0);
  const user = ref(null);
  const loading = ref(false);

  onMounted(() => {
    if (!code) return;
    loading.value = true;
    axios.post(`${import.meta.env.VITE_AUTH_ENDPOINT}/login`, {
      code
    }).then(res => {
      accessToken.value = res.data.accessToken;
      refreshToken.value = res.data.refreshToken;
      expiresIn.value = res.data.expiresIn;
      emit('onAuthChanged', accessToken.value);
      window.history.pushState({}, null, '/tempothing');
      loading.value = false;
    }).catch(e => {
      window.history.pushState({}, null, '/tempothing');
      loading.value = false;
    });
  });

  watch([refreshToken, expiresIn], async () => {
    if (!refreshToken.value || !expiresIn.value) return;
    const refreshInterval = setInterval(async () => {
        axios.post(`${import.meta.env.VITE_AUTH_ENDPOINT}/refresh`, {
          code
        }).then(res => {
          accessToken.value = res.data.accessToken;
          expiresIn.value = res.data.expiresIn;
        });
    }, (expiresIn.value - 60) * 1000);
    return () => clearInterval(refreshInterval);
  }, { immediate: true });

  watch(accessToken, () => {
    if (!accessToken.value) return;
    spotifyApi.setAccessToken(accessToken.value);
    spotifyApi.getMe().then(res => {
      user.value = {
        name: res.body.display_name
      };
    });
  }, { immediate: true });
</script>

<template>
  <div v-if="!loading">
    <a v-if="!accessToken" :href="AUTH_URL" class="no-select">Login</a>
    <p v-if="accessToken && user">{{ user.name }}</p>
  </div>
  <div v-if="loading" class="spinner small-spinner"></div>
</template>
