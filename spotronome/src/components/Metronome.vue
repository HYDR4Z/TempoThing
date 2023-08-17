<script setup>
  import { ref, watch } from 'vue';

  const tempo = new ref(60);
  const volume = new ref(50);
  const beatFreq = new ref(1000);
  const quarterFreq = new ref(800);
  
  let audioContext;
  let isRunning = false;
  let currentBeatInBar = 0;
  let nextNoteTime = 0.0;
  let lookahead = 25;
  let intervalID = null;
  let scheduleAheadTime = 0.1;
  let currentQuarterNote = 0;

  const increaseTempo = () => {
    tempo.value += 1;
  }

  const decreaseTempo = () => {
    tempo.value -= 1;
  }

  const scheduleNote = (beatNumber, time) => {
    const osc = audioContext.createOscillator();
    const envelope = audioContext.createGain();
        
    osc.frequency.value = (beatNumber % 4 == 0) ? beatFreq.value : quarterFreq.value;

    envelope.gain.exponentialRampToValueAtTime(volume.value / 100, time + 0.001);
    envelope.gain.exponentialRampToValueAtTime(0.001, time + 0.02);

    osc.connect(envelope);
    envelope.connect(audioContext.destination);
    
    osc.start(time);
    osc.stop(time + 0.03);
  }

  const scheduler = () => {
    // while there are notes that will need to play before the next interval, schedule them and advance the pointer.
    while (nextNoteTime < audioContext.currentTime + scheduleAheadTime ) {
      scheduleNote(currentQuarterNote, nextNoteTime);

      // Advance current note and time by a quarter note (crotchet if you're posh)
      nextNoteTime += (60.0 / tempo.value); // Add beat length to last beat time
  
      currentQuarterNote++;
      if (currentQuarterNote == 4) {
          currentQuarterNote = 0;
      }
    }
  }

  const startStop = () => {
    if (!isRunning) {
      if (audioContext == null) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
      }
      isRunning = true;
      currentBeatInBar = 0;
      nextNoteTime = audioContext.currentTime + 0.05;
      intervalID = setInterval(() => scheduler(), lookahead);
    } else {
      isRunning = false;
      clearInterval(intervalID);
    }
  }
</script>

<template>
  <div class="metronome-wrapper">
    <div class="metronome-section">
      <a @click="startStop">

      </a>
    </div>
    <div class="metronome-section">
      <div class="tempo-input-wrapper">
        <i class="fa-solid fa-minus fa-fw" @click="decreaseTempo"></i>
        <input class="tempo-input" type="number" v-model="tempo" @click="startStop" />
        <i class="fa-solid fa-plus fa-fw" @click="increaseTempo"></i>
      </div>
      <div class="volume-input-wrapper">
        <i class="fa-solid fa-volume-off fa-fw"></i>
        <input type="range" min="0" max="100" class="tone-slider" v-model="volume" />
        <i class="fa-solid fa-volume-high fa-fw"></i>
      </div>
    </div>
    <div class="metronome-section">
      <p>Tone</p>
      <input type="range" min="100" max="2500" class="tone-slider" v-model="beatFreq" />
      <input type="range" min="100" max="2500" class="tone-slider" v-model="quarterFreq" />
    </div>
  </div>
</template>
