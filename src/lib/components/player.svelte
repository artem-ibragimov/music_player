<script lang="ts">
   import { createEventDispatcher } from 'svelte';
   import { formatSecs } from '../util/tools';

   export let trackName: string;
   let audio: HTMLAudioElement;
   const playbackSpeed = [0.5, 0.75, 0.9, 1, 1.25, 1.5];
   let selectedSpeed = 1;
   let error: Error;
   let formatedDuration = '';
   let formatedCurrentTime = '';
   let currentTime = 0;
   let isPaused = true;

   let startRange: number | '' = '';
   let endRange: number | '' = '';

   $: formatedCurrentTime = formatSecs(currentTime);
   $: if (audio) {
      audio.playbackRate = selectedSpeed;
   }

   const dispatcher = createEventDispatcher();

   function onPlay() {
      isPaused ? audio.play() : audio.pause();
      isPaused = audio.paused;
   }

   function onSeek({ currentTarget }: { currentTarget: EventTarget & HTMLInputElement }) {
      currentTime = Number(currentTarget.value);
      audio.currentTime = currentTime;
   }

   function onTimeUpdate() {
      currentTime = audio.currentTime;
      if (audio.currentTime === audio.duration) {
         dispatcher('end');
      }
      checkRange();
   }

   function checkRange() {
      if (endRange != '' && audio.currentTime > endRange) {
         audio.currentTime = startRange || 0;
      }
   }

   function onMetaLoaded() {
      formatedDuration = formatSecs(audio.duration);
   }

   function setStartRange() {
      if (startRange !== '') {
         startRange = '';
         return;
      }
      startRange = Math.floor(currentTime);
   }

   function setEndRange() {
      if (endRange !== '') {
         endRange = '';
         return;
      }
      endRange = Math.ceil(currentTime);
   }
</script>

{#if !!trackName}
   <audio bind:this={audio} on:loadeddata={onMetaLoaded} on:timeupdate={onTimeUpdate}>
      <source src={`/static/music/${trackName}.mp3`} type="audio/mpeg" />
      Your browser does not support the audio element.
   </audio>
{/if}

<div class="Player">
   {#if !!trackName}
      <div class="Player__trackName">{trackName}</div>
   {/if}
   {#if !!error}
      <div class="Player__error">{`${error}`}</div>
   {/if}

   <div class="Player_row">
      <span class="time">{formatedCurrentTime}</span>
      <input type="range" max={audio && audio.duration} value={currentTime} on:input={onSeek} />
      <span class="time">{formatedDuration}</span>
   </div>

   <div class="Player_row">
      <div class="Player__button" class:Player__button-active={startRange !== ''} on:click={setStartRange}>
         {'{ ' + startRange}
      </div>
      <div class="Player__button" class:Player__button-active={endRange !== ''} on:click={setEndRange}>
         {endRange + ' }'}
      </div>

      <select bind:value={selectedSpeed} class="Player__button">
         {#each playbackSpeed as speed}
            <option selected={selectedSpeed === speed}>{speed}</option>
         {/each}
      </select>

      <div on:click={onPlay} class="Player__button">{isPaused ? '  >  ' : '  ||  '}</div>
   </div>
</div>

<style scoped>
   .Player {
      position: fixed;
      right: 0;
      left: 0;
      bottom: 0;
      display: flex;
      flex-direction: column;
   }
   .Player_row {
      justify-content: space-between;
      display: flex;
      padding: 5px;
   }
   .Player__button {
      cursor: pointer;
      padding: 15px 25px;
      border-radius: 6px;
      border: 1px solid #ccc;
   }
   .Player__button-active {
      background: whitesmoke;
   }
   .Player__trackName {
   }
   .Player__error {
      background: rgb(168, 0, 0);
      color: whitesmoke;
   }

   input[type='range'] {
      flex: 1;
      -webkit-appearance: none;
   }
   input[type='range']::-webkit-slider-runnable-track {
      width: 100%;
      height: 3px;
      cursor: pointer;
      background: linear-gradient(to right, rgba(0, 125, 181, 0.6), rgba(0, 125, 181, 0.2));
   }
   input[type='range']::before {
      position: absolute;
      content: '';
      top: 8px;
      left: 0;
      width: var(--seek-before-width);
      height: 3px;
      background-color: #007db5;
      cursor: pointer;
   }
   input[type='range']::-webkit-slider-thumb {
      position: relative;
      -webkit-appearance: none;
      box-sizing: content-box;
      border: 1px solid #007db5;
      height: 15px;
      width: 15px;
      border-radius: 50%;
      background-color: #fff;
      cursor: pointer;
      margin: -7px 0 0 0;
   }
   input[type='range']:active::-webkit-slider-thumb {
      transform: scale(1.2);
      background: #007db5;
   }
   input[type='range']::-moz-range-track {
      width: 100%;
      height: 3px;
      cursor: pointer;
      background: linear-gradient(
         to right,
         rgba(0, 125, 181, 0.6) var(--buffered-width),
         rgba(0, 125, 181, 0.2) var(--buffered-width)
      );
   }
   input[type='range']::-moz-range-progress {
      background-color: #007db5;
   }
   input[type='range']::-moz-focus-outer {
      border: 0;
   }
   input[type='range']::-moz-range-thumb {
      box-sizing: content-box;
      border: 1px solid #007db5;
      height: 15px;
      width: 15px;
      border-radius: 50%;
      background-color: #fff;
      cursor: pointer;
   }
   input[type='range']:active::-moz-range-thumb {
      transform: scale(1.2);
      background: #007db5;
   }
   input[type='range']::-ms-track {
      width: 100%;
      height: 3px;
      cursor: pointer;
      background: transparent;
      border: solid transparent;
      color: transparent;
   }
   input[type='range']::-ms-fill-lower {
      background-color: #007db5;
   }
   input[type='range']::-ms-fill-upper {
      background: linear-gradient(
         to right,
         rgba(0, 125, 181, 0.6) var(--buffered-width),
         rgba(0, 125, 181, 0.2) var(--buffered-width)
      );
   }
   input[type='range']::-ms-thumb {
      box-sizing: content-box;
      border: 1px solid #007db5;
      height: 15px;
      width: 15px;
      border-radius: 50%;
      background-color: #fff;
      cursor: pointer;
   }
   input[type='range']:active::-ms-thumb {
      transform: scale(1.2);
      background: #007db5;
   }
</style>
