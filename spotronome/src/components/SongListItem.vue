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
  <article class="track-list-item">
    <img :src="track.albumUrl" />
    <div>
      <div class="track-list-item-details">
        <p class="track-list-item-title">{{ track.title }}</p>
        <p class="track-list-item-artist">{{ track.artist.name }}</p>
      </div>
      <div class="track-list-item-actions">
        <i class="fa-brands fa-spotify fa-fw" @click="playTrackSpotify"></i>
        <i class="fa-solid fa-play fa-fw" @click="playTrack"></i>
      </div>
    </div>
  </article>
</template>
