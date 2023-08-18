<script setup>
  import SongSearch from './components/SongSearch.vue';
  import PlaylistBrowser from './components/PlaylistBrowser.vue';
  import Login from './components/Login.vue';
  import Metronome from './components/Metronome.vue';
  import Player from './components/Player.vue';
  import { ref } from 'vue';

  const accessToken = ref(null);
  const metronome = ref(null);
  const player = ref(null);

  const onAuthChanged = (token) => {
    accessToken.value = token;
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
    <h1>TempoThing</h1>
    <div class="nav-wrapper">
      <p></p>
      <Login v-if="!accessToken" @onAuthChanged="onAuthChanged" />
    </div>
  </header>
  <main>
    <div class="home-wrapper" v-if="accessToken">
      <PlaylistBrowser :token="accessToken" />
      <SongSearch :token="accessToken" @onTrackPlay="onTrackPlay" />
    </div>
    <h2 class="signin-text" v-if="!accessToken">
      <a>Sign in using Spotify</a> to sync to your music
    </h2>
  </main>
  <footer>
    <Player ref="player" :token="accessToken" />
    <Metronome ref="metronome" />
  </footer>
</template>
