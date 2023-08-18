<script setup>
  import { ref, toRefs, watch } from 'vue';
  import SpotifyWebApi from 'spotify-web-api-node';
  import SongListItem from './SongListItem.vue';
  
  const emit = defineEmits(['onTrackPlay']);

  const props = defineProps({
    token: String
  });

  const { token } = toRefs(props);
  const searchText = ref('');
  const searchResults = ref([]);

  const spotifyApi = new SpotifyWebApi({
    clientId: 'dde63debce154f3f85fa86c5b6e43ddb'
  });

  watch(token, () => {
    if (token.value) spotifyApi.setAccessToken(token.value);
  }, { immediate: true });

  watch([searchText, token], () => {
    if (!searchText.value || !token.value) {
      return searchResults.value = [];
    }
    spotifyApi.searchTracks(searchText.value).then(res => {
      searchResults.value = res.body.tracks.items.map(track => {
        const smallestImage = track.album.images.reduce((smallest, image) => {
          if (image.height < smallest.height) return image;
          return smallest;
        });

        return {
          id: track.id,
          title: track.name,
          url: track.external_urls.spotify,
          artist: {
            name: track.artists[0].name,
            url: track.artists[0].external_urls.spotify,
          },
          album: {
            image: smallestImage.url
          }
        };
      });
    });
  });

  const onTrackPlay = (trackId, listen) => {
    spotifyApi.getAudioFeaturesForTrack(trackId).then(res => {
      emit('onTrackPlay', res.body, listen)
    });
  }
</script>

<template>
  <div class="song-search-wrapper">
    <input class="box-input search-input" type="text" v-model="searchText" placeholder="Enter a song or artist..." />
    <div class="result-list">
      <SongListItem v-for="track in searchResults" :track="track" @onTrackPlay="onTrackPlay" />
    </div>
  </div>
</template>
