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
        <a class="track-list-item-title" target="_blank" :href="`https://open.spotify.com/track/${track.id}`">{{ track.title }}</a>
        <a class="track-list-item-artist" target="_blank" :href="track.artist.external_urls.spotify">{{ track.artist.name }}</a>
      </div>
      <div class="track-list-item-actions">
        <i class="fa-brands fa-spotify fa-fw" @click="playTrackSpotify"></i>
        <i class="fa-solid fa-play fa-fw" @click="playTrack"></i>
      </div>
    </div>
  </article>
</template>
