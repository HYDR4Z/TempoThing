// https://grantjam.es/creating-a-simple-metronome-using-javascript-and-the-web-audio-api/
<script setup>
  import { ref, watch } from 'vue';

  const tempo = new ref(120);
  const volume = new ref(25);
  const beatFreq = new ref(1108.73);
  const quarterFreq = new ref(880);

  const increaseTempo = () => {
    tempo.value += 1;
  }

  const decreaseTempo = () => {
    tempo.value -= 1;
  }

  const isRunning = ref(false);
  let audioContext = null;
  let notesInQueue = [];
  let currentBeatInBar = 0;
  let beatsPerBar = 4;
  let lookahead = 0;
  let scheduleAheadTime = 0.1;
  let nextNoteTime = 0.0;
  let intervalID = null;

  const nextNote = () => {
    var secondsPerBeat = 60.0 / tempo.value;
    nextNoteTime += secondsPerBeat;

    currentBeatInBar++;
    if (currentBeatInBar == beatsPerBar) currentBeatInBar = 0;
  }

  const scheduleNote = (beatNumber, time) => {
    notesInQueue.push({ note: beatNumber, time: time });

    const osc = audioContext.createOscillator();
    const envelope = audioContext.createGain();
    
    osc.frequency.value = (beatNumber % beatsPerBar == 0) ? beatFreq.value : quarterFreq.value;
    envelope.gain.value = 1;
    envelope.gain.exponentialRampToValueAtTime(volume.value / 100, time + 0.001);
    envelope.gain.exponentialRampToValueAtTime(0.001, time + 0.02);

    osc.connect(envelope);
    envelope.connect(audioContext.destination);

    osc.start(time);
    osc.stop(time + 0.03);
  }

  const scheduler = () => {
    while (nextNoteTime < audioContext.currentTime + scheduleAheadTime ) {
      scheduleNote(currentBeatInBar, nextNoteTime);
      nextNote();
    }
  }

  const start = () => {
    if (isRunning) stop();
    if (audioContext == null) audioContext = new (window.AudioContext || window.webkitAudioContext)();
    isRunning.value = true;
    currentBeatInBar = 0;
    nextNoteTime = audioContext.currentTime + 0.05;
    intervalID = setInterval(() => scheduler(), lookahead);
  }

  const stop = () => {
    isRunning.value = false;
    clearInterval(intervalID);
  }

  const startStop = () => {
    if (!isRunning.value) {
      start();
    } else {
      stop();
    }
  }

  const setTrackInfo = (newTempo, newTimeSignature) => {
    tempo.value = newTempo;
    beatsPerBar = newTimeSignature;
  }

  defineExpose({
    start,
    setTrackInfo
  });
</script>

<template>
  <div class="metronome-wrapper">
    <a class="playpause-button" @click="startStop">
      <i v-if="!isRunning" class="fa-solid fa-play fa-fw"></i>
      <i v-if="isRunning" class="fa-solid fa-pause fa-fw"></i>
    </a>
    <div class="metronome-section">
      <div class="tempo-input-wrapper">
        <i class="fa-solid fa-minus fa-fw" @click="decreaseTempo"></i>
        <input class="box-input" type="number" v-model="tempo" />
        <i class="fa-solid fa-plus fa-fw" @click="increaseTempo"></i>
      </div>
      <div class="volume-input-wrapper">
        <i class="fa-solid fa-volume-off fa-fw"></i>
        <input type="range" min="0" max="100" class="volume-slider" v-model="volume" />
        <i class="fa-solid fa-volume-high fa-fw"></i>
      </div>
    </div>
    <div class="metronome-section">
      <p>Tone</p>
      <input type="range" min="110" max="3520" class="tone-slider" v-model="beatFreq" />
      <input type="range" min="100" max="3520" class="tone-slider" v-model="quarterFreq" />
    </div>
  </div>
</template>
