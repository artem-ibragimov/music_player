<script lang="ts">
   import { tracksList } from '../store/tracks.store';
   import Player from '../components/player.svelte';

   let trackName: string;
   let query: string;

   $: reg = new RegExp(query, 'gi');
   $: filteredTracks = query ? $tracksList.filter((t) => reg.test(t)) : $tracksList;

   function selectTrack(e) {
      trackName = e.target.innerText;
   }
</script>

<div on:click={selectTrack} class="TrackList">
   <input type="search" bind:value={query} placeholder="Search"/>

   {#each filteredTracks as track}
      <h3>{track}</h3>
   {/each}
</div>
<Player {trackName} />

<style scoped>
   .TrackList {
      display: flex;
      flex-direction: column;
      cursor: pointer;
   }
</style>
