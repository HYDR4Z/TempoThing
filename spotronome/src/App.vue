<script setup>
  import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <h1>Tempothing</h1>
    <div class="nav-wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
      </nav>
      <a>Logout</a>
    </div>
  </header>

  <main>
    <RouterView />
  </main>
</template>
