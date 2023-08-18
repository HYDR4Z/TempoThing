<script setup>
  import { toRefs } from 'vue';

  const emit = defineEmits(['onTrackPlay']);

  const props = defineProps({
    track: Object
  });

  const { track } = toRefs(props);

  const playTrack = () => {
    emit('onTrackPlay', track.value.id, false);
  }

  const playTrackSpotify = () => {
    emit('onTrackPlay', track.value.id, true);
  }
</script>

<template>
  <article class="result-list-item">
    <img :src="track.album.image" />
    <div>
      <div class="result-list-item-details">
        <a class="result-list-item-primary" target="_blank" :href="track.url">{{ track.title }}</a>
        <a class="result-list-item-secondary" target="_blank" :href="track.artist.url">{{ track.artist.name }}</a>
      </div>
      <div class="result-list-item-actions">
        <i class="fa-brands fa-spotify fa-fw" @click="playTrackSpotify"></i>
        <i class="fa-solid fa-play fa-fw" @click="playTrack"></i>
      </div>
    </div>
  </article>
</template>
