<script setup>
  import SongSearch from './components/SongSearch.vue';
  import Login from './components/Login.vue';
  import Metronome from './components/Metronome.vue';
  import { ref } from 'vue';

  const accessToken = ref(null);
  const metronome = ref(null)

  const onAuthChanged = (token) => {
    accessToken.value = token;
  }

  const onTrackPlay = (track) => {
    console.log(track.tempo)
    metronome.value.setTempo(track.tempo);
    metronome.value.start();
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
    <div class="home-wrapper">
      <SongSearch :token="accessToken" @onTrackPlay="onTrackPlay" />
    </div>
  </main>
  <footer>
    <p></p>
    <Metronome ref="metronome" />
  </footer>
</template>
