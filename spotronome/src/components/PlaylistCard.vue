<script setup>
  import { toRefs } from 'vue';

  const emit = defineEmits(['onSelectPlaylist']);

  const props = defineProps({
    playlist: Object
  });

  const { playlist } = toRefs(props);

  const selectPlaylist = () => {
    emit('onSelectPlaylist', playlist.value);
  }
</script>

<template>
  <article class="result-list-item">
    <img :src="playlist.album.image" />
    <div>
      <div class="result-list-item-details">
        <a class="result-list-item-primary" target="_blank" :href="playlist.url">{{ playlist.title }}</a>
        <a class="result-list-item-secondary" target="_blank" :href="playlist.owner.url">{{ playlist.owner.name }}</a>
      </div>
      <div class="result-list-item-actions">
        <i class="fa-solid fa-arrow-right fa-fw" @click="selectPlaylist"></i>
      </div>
    </div>
  </article>
</template>
