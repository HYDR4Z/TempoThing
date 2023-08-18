<script setup>
  import SpotifyWebApi from 'spotify-web-api-node';
  import { onMounted, ref, toRefs, watch } from 'vue';

  const props = defineProps({
    token: String
  });

  const { token } = toRefs(props);
  const playbackState = ref(null);

  const spotifyApi = new SpotifyWebApi({
    clientId: 'dde63debce154f3f85fa86c5b6e43ddb'
  });

  watch(token, () => {
    if (token.value) spotifyApi.setAccessToken(token.value);
  }, { immediate: true });

  const playTrack = async (trackId) => {
    return await spotifyApi.play({ uris: [`spotify:track:${trackId}`] });
  }

  onMounted(() => {
    setInterval(() => {
      if (!token.value) return;
      spotifyApi.getMyCurrentPlaybackState().then(res => {
        const smallestImage = res.body.item.album.images.reduce((smallest, image) => {
          if (image.height < smallest.height) return image;
          return smallest;
        });
        playbackState.value = {
          isPlaying: res.body.is_playing,
          progress: Math.floor(res.body.progress_ms / 1000),
          track: {
            id: res.body.item.id,
            length: Math.floor(res.body.item.duration_ms / 1000),
            artist: res.body.item.artists[0],
            title: res.body.item.name,
            albumUrl: smallestImage.url
          }
        };
      });
    }, 500);
  });

  const startStop = () => {
    if (playbackState.isPlaying) spotifyApi.pause();
    else spotifyApi.play();
  }

  defineExpose({
    playTrack
  });
</script>

<template>
  <div class="player-wrapper" v-if="playbackState">
    <img :src="playbackState.track.albumUrl" />
    <div>
      <div class="track-list-item-details">
        <p class="track-list-item-title">{{ playbackState.track.title }}</p>
        <p class="track-list-item-artist">{{ playbackState.track.artist.name }}</p>
      </div>
      <div class="track-list-item-details">
        <input type="range" min="0" v-bind:max="playbackState.track.length" class="tone-slider" v-model="playbackState.progress" />
      </div>
    </div>
    <a class="playpause-button" @click="startStop">
      <i v-if="!playbackState.isPlaying" class="fa-solid fa-play fa-fw"></i>
      <i v-if="playbackState.isPlaying" class="fa-solid fa-pause fa-fw"></i>
    </a>
  </div>
</template>
