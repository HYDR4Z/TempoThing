<script setup>
  import { RouterLink, RouterView } from 'vue-router';
  import Login from './components/Login.vue';
  import { ref } from 'vue';

  const accessToken = ref('');

  const onAuthChanged = (token) => {
    console.log(token)
    accessToken.value = token;
  }

  const log = () => {
    console.log(!accessToken.value);
  }
</script>

<template>
  <header>
    <h1>TempoThing</h1>
    <div class="nav-wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
      </nav>
      <Login v-if="!accessToken.value" @onAuthChanged="onAuthChanged" /> <!-- Use 'show.value' here -->
    </div>
  </header>
  <main>
    <h1 @click="log">click</h1>
    <RouterView />
  </main>
</template>
