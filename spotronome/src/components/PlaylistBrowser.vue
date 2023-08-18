<script setup>
  import { onMounted, ref, toRefs, watch } from 'vue';
  import SpotifyWebApi from 'spotify-web-api-node';
  import PlaylistCard from './PlaylistCard.vue';

  const props = defineProps({
    token: String
  });

  const { token } = toRefs(props);
  const playlists = ref([]);

  const spotifyApi = new SpotifyWebApi({
    clientId: 'dde63debce154f3f85fa86c5b6e43ddb'
  });

  watch(token, () => {
    if (token.value) spotifyApi.setAccessToken(token.value);
  }, { immediate: true });

  onMounted(() => {
    if (!token.value) return;
    spotifyApi.getUserPlaylists({ limit: 50 }).then(res => {
      playlists.value = res.body.items.map(playlist => {
        return {
          id: playlist.id,
          title: playlist.name,
          url: playlist.external_urls.spotify,
          owner: {
            name: playlist.owner.display_name,
            url: playlist.owner.external_urls.spotify,
          },
          album: {
            image: playlist.images[0].url
          }
        };
      });
    });
  });
</script>

<template>
  <div class="song-search-wrapper">
    <input class="box-input search-input" type="text" value="Playlists" disabled />
    <div class="result-list">
      <PlaylistCard v-for="playlist in playlists" :playlist="playlist" />
    </div>
  </div>
</template>
