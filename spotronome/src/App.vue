<script setup>
  import { RouterLink, RouterView } from 'vue-router';
  import Login from './components/Login.vue';
  import Metronome from './components/Metronome.vue';
  import { ref } from 'vue';

  const accessToken = ref('');

  const onAuthChanged = (token) => {
    accessToken.value = token;
  }
</script>

<template>
  <header>
    <h1>TempoThing</h1>
    <div class="nav-wrapper">
      <nav>
        <!-- <RouterLink to="/">Home</RouterLink> -->
      </nav>
      <Login v-if="!accessToken" @onAuthChanged="onAuthChanged" /> <!-- Use 'show.value' here -->
    </div>
  </header>
  <main>
    <RouterView />
  </main>
  <footer>
    <p></p>
    <Metronome />
  </footer>
</template>
