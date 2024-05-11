<script>
  import { writable } from 'svelte/store';
  import Graph1 from '$lib/Graph1.svelte'; // Import the Graph1 component
  import { setContext } from 'svelte'



  // Initialize the selected file name as a writable store
  let selectedFileName = writable('');
  // $: fileName = "Forcast_ActualSales2024";
// Function to handle button click and set the selected file name
function selectFileName(fileName) {
  selectedFileName.set(fileName); // Set the value of the writable store
}
</script>

<div>
  <!-- Button to select "Forcast_ActualSales2022" file name -->
  <button on:click={() => selectFileName("Forcast_ActualSales2022")}>Load 2022 Data</button>
  <!-- Button to select "Forcast_ActualSales2023" file name -->
  <button on:click={() => selectFileName("Forcast_ActualSales2023")}>Load 2023 Data</button>
  <!-- Button to select "Forcast_ActualSales2024" file name -->
  <button on:click={() => selectFileName("Forcast_ActualSales2024")}>Load 2024 Data</button>


  {#if $selectedFileName === "Forcast_ActualSales2022"}
    <!-- <input bind:value={$selectedFileName} /> -->
    <Graph1 fileName="Forcast_ActualSales2022" />
  {:else if $selectedFileName === "Forcast_ActualSales2023"}
    <!-- <input bind:value={$selectedFileName} /> -->
    <Graph1 fileName="Forcast_ActualSales2023" />
  {:else if $selectedFileName === "Forcast_ActualSales2024"}
    <!-- <input bind:value={$selectedFileName} /> -->
    <Graph1 fileName="Forcast_ActualSales2024" />
  {/if}
</div>
