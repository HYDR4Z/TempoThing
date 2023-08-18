<script setup>
  import { onMounted, ref, toRefs, watch } from 'vue';
  import SpotifyWebApi from 'spotify-web-api-node';
  import PlaylistCard from './PlaylistCard.vue';
  import SongCard from './SongCard.vue';

  const emit = defineEmits(['onTrackPlay']);

  const props = defineProps({
    token: String
  });

  const spotifyApi = new SpotifyWebApi({
    clientId: 'dde63debce154f3f85fa86c5b6e43ddb'
  });

  const { token } = toRefs(props);
  const playlists = ref([]);
  const isLoading = ref(false);
  const selectedPlaylist = ref(null);

  watch(token, () => {
    if (token.value) spotifyApi.setAccessToken(token.value);
  }, { immediate: true });

  const onSelectPlaylist = (playlist) => {
    isLoading.value = true;
    spotifyApi.getPlaylistTracks(playlist.id, { limit: 100 }).then(res => {
      selectedPlaylist.value = { 
        ...playlist, 
        tracks: res.body.items.map(item => {
          const track = item.track;
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
        })
      };
      isLoading.value = false;
    });
  }

  const deselectPlaylist = () => {
    selectedPlaylist.value = null;
  }

  const onTrackPlay = (trackId, listen) => {
    spotifyApi.getAudioFeaturesForTrack(trackId).then(res => {
      emit('onTrackPlay', res.body, listen)
    });
  }

  onMounted(() => {
    if (!token.value) return;
    isLoading.value = true;
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
    isLoading.value = false;
  });
</script>

<template>
  <section>
    <div class="result-list-header">
      <i class="fa-solid fa-arrow-left fa-fw" v-if="selectedPlaylist" @click="deselectPlaylist"></i>
      {{ !selectedPlaylist ? 'Playlists' : selectedPlaylist.title }}
    </div>
    <div class="spinner" v-if="isLoading"></div>
    <div class="result-list" v-if="!selectedPlaylist && !isLoading">
      <PlaylistCard v-for="playlist in playlists" :playlist="playlist" @onSelectPlaylist="onSelectPlaylist" />
      <h3 class="result-list-info" v-if="playlists.length == 50">This list only shows up to 50 items...</h3>
    </div>
    <div class="result-list" v-if="selectedPlaylist && !isLoading">
      <SongCard v-for="track in selectedPlaylist.tracks" :track="track" @onTrackPlay="onTrackPlay" />
      <h3 class="result-list-info" v-if="selectedPlaylist.tracks.length == 100">This list only shows up to 100 items...</h3>
    </div>
  </section>
</template>
