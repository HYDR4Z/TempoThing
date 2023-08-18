<script setup>
  import { onMounted, ref, watch } from 'vue';
  import axios from 'axios';
  import SpotifyWebApi from 'spotify-web-api-node';

  const emit = defineEmits(['onAuthChanged']);

  const spotifyApi = new SpotifyWebApi({
    clientId: 'dde63debce154f3f85fa86c5b6e43ddb'
  });

  const AUTH_URL = "https://accounts.spotify.com/authorize?" + 
  new URLSearchParams({
    response_type: 'code',
    client_id: 'dde63debce154f3f85fa86c5b6e43ddb',
    scope: 'user-read-email user-library-read user-read-playback-state user-modify-playback-state playlist-read-private user-read-private user-read-email',
    redirect_uri: 'http://localhost:5173'
  }).toString();

  const code = new URLSearchParams(window.location.search).get('code');

  const accessToken = ref('');
  const refreshToken = ref('');
  const expiresIn = ref(0);
  const user = ref(null);

  onMounted(() => {
    if (!code) return;
    axios.post(`${import.meta.env.VITE_AUTH_ENDPOINT}/login`, {
      code
    }).then(res => {
      accessToken.value = res.data.accessToken;
      refreshToken.value = res.data.refreshToken;
      expiresIn.value = res.data.expiresIn;
      emit('onAuthChanged', accessToken.value);
      window.history.pushState({}, null, '/');
    })
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
  <a v-if="!accessToken" :href="AUTH_URL">Login</a>
  <p v-if="accessToken && user">{{ user.name }}</p>
</template>
