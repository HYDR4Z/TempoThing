<script setup>
  import Songs from './components/Songs.vue';
  import Playlists from './components/Playlists.vue';
  import Auth from './components/Auth.vue';
  import Metronome from './components/Metronome.vue';
  import Player from './components/Player.vue';
  import { ref } from 'vue';

  const accessToken = ref(null);
  const metronome = ref(null);
  const player = ref(null);

  const onAuthChanged = (token) => {
    accessToken.value = token;
  }

  const onSyncMetronome = (track) => {
    metronome.value.setTrackInfo(track.tempo, track.time_signature);
  }

  const onTrackPlay = (track, listen) => {
    metronome.value.setTrackInfo(track.tempo, track.time_signature);
    if (listen) {
      player.value.playTrack(track.id).then(() => {
        metronome.value.start();
      });
    } else {
      metronome.value.start();
    }
  }
</script>

<template>
  <header>
    <h1 class="no-select">TempoThing</h1>
    <div class="nav-wrapper">
      <p></p>
      <Auth @onAuthChanged="onAuthChanged" />
    </div>
  </header>
  <main>
    <div class="main-wrapper" v-if="accessToken">
      <Playlists :token="accessToken" @onTrackPlay="onTrackPlay" />
      <Songs :token="accessToken" @onTrackPlay="onTrackPlay" />
    </div>
    <h2 class="signin-text" v-if="!accessToken">
      Sign in using Spotify to sync to your music
    </h2>
  </main>
  <footer>
    <Player ref="player" :token="accessToken" @onSyncMetronome="onSyncMetronome" />
    <Metronome ref="metronome" />
  </footer>
</template>
