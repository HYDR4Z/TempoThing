<script setup>
  import SpotifyWebApi from 'spotify-web-api-node';
  import { toRefs, watch } from 'vue';

  const props = defineProps({
    token: String
  });

  const { token } = toRefs(props);

  const spotifyApi = new SpotifyWebApi({
    clientId: 'dde63debce154f3f85fa86c5b6e43ddb'
  });

  watch(token, () => {
    if (token.value) spotifyApi.setAccessToken(token.value);
  }, { immediate: true });

  const playTrack = async (trackId) => {
    return await spotifyApi.play({ uris: [`spotify:track:${trackId}`] });
  }

  defineExpose({
    playTrack
  });
</script>

<template>
  <div class="player-wrapper">

  </div>
</template>
