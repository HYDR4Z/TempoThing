<script setup>
  import SpotifyWebApi from 'spotify-web-api-node';
  import { onMounted, ref, toRefs, watch } from 'vue';

  const emit = defineEmits(['onSyncMetronome']);

  const props = defineProps({
    token: String
  });

  const { token } = toRefs(props);
  const playbackState = ref(null);
  const draggingTimebar = ref(false);

  const spotifyApi = new SpotifyWebApi({
    clientId: 'dde63debce154f3f85fa86c5b6e43ddb'
  });

  watch(token, () => {
    if (token.value) spotifyApi.setAccessToken(token.value);
  }, { immediate: true });

  const playTrack = async (trackId) => {
    return await spotifyApi.play({ uris: [`spotify:track:${trackId}`] }).then(() => fetchPlaybackState());
  }

  const fetchPlaybackState = () => {
    if (!token.value) return;
    spotifyApi.getMyCurrentPlaybackState().then(res => {
      if (!res.body || !res.body.item) return;
      const smallestImage = res.body.item.album.images.reduce((smallest, image) => {
        if (image.height < smallest.height) return image;
        return smallest;
      });
      if (draggingTimebar.value) return;
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
  }

  onMounted(() => {
    setInterval(() => {
      if (draggingTimebar.value) return;
      fetchPlaybackState();
    }, 500);
  });

  const startStop = () => {
    if (playbackState.value.isPlaying) {
      spotifyApi.pause().then(() => {
        fetchPlaybackState();
      });
    } else {
      spotifyApi.play().then(() => {
        fetchPlaybackState();
      });
    }
  }

  const previousSong = () => {
    spotifyApi.skipToPrevious();
  }

  const skipSong = () => {
    spotifyApi.skipToNext();
  }

  const syncMetronome = () => {
    spotifyApi.getAudioFeaturesForTrack(playbackState.value.track.id).then(res => {
      emit('onSyncMetronome', res.body);
    });
  }

  const onProgressInput = () => {
    draggingTimebar.value = true;
  }

  const setProgress = (e) => {
    spotifyApi.seek(e.target.value * 1000).then(res => {
      draggingTimebar.value = true;
    });
  }

  defineExpose({
    playTrack
  });
</script>

<template>
  <div v-if="!playbackState && token"></div>
  <div class="player-wrapper" v-if="playbackState">
    <img class="player-image" :src="playbackState.track.albumUrl" />
    <a class="image-playpause-overlay" @click="startStop">
      <i v-if="!playbackState.isPlaying" class="fa-solid fa-play fa-fw"></i>
      <i v-if="playbackState.isPlaying" class="fa-solid fa-pause fa-fw"></i>
    </a>
    <div class="player-track-details">
      <a class="player-track-title" target="_blank" :href="`https://open.spotify.com/track/${playbackState.track.id}`">{{ playbackState.track.title }}</a>
      <a class="player-track-artist" target="_blank" :href="playbackState.track.artist.external_urls.spotify">{{ playbackState.track.artist.name }}</a>
    </div>
    <div class="player-section">
      <div class="player-control-buttons">
        <i class="fa-solid fa-computer fa-fw"></i>
        <i class="fa-solid fa-backward fa-fw" @click="previousSong"></i>
        <a class="player-playpause" @click="startStop">
          <i v-if="!playbackState.isPlaying" class="fa-solid fa-play fa-fw"></i>
          <i v-if="playbackState.isPlaying" class="fa-solid fa-pause fa-fw"></i>
        </a>
        <i class="fa-solid fa-forward fa-fw" @click="skipSong"></i>
        <i class="fa-solid fa-rotate fa-fw" @click="syncMetronome"></i>
      </div>
      <input type="range" min="0" :max="playbackState.track.length" class="tone-slider" @input="onProgressInput" @change="setProgress" v-model="playbackState.progress" />
    </div>
  </div>
</template>
