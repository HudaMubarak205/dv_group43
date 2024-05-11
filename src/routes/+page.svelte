<script>
  import { createEventDispatcher } from 'svelte';
  import Graph1 from '$lib/Graph1.svelte'; // Import the Graph1 component

  const dispatch = createEventDispatcher(); // Event dispatcher to communicate with parent components

  let selectedFileName = ''; // Initialize the selected file name variable

  // Function to handle button click and set the selected file name
  function selectFileName(fileName) {
    selectedFileName = fileName;
    dispatch('fileSelected', { fileName }); // Dispatch event to communicate with parent components
  }
</script>

<div>
  <!-- Button to select "Forcast_ActualSales2022" file name -->
  <button on:click={() => selectFileName("Forcast_ActualSales2022")}>Load 2022 Data</button>
  <!-- Button to select "Forcast_ActualSales2023" file name -->
  <button on:click={() => selectFileName("Forcast_ActualSales2023")}>Load 2023 Data</button>
  <!-- Button to select "Forcast_ActualSales2024" file name -->
  <button on:click={() => selectFileName("Forcast_ActualSales2024")}>Load 2024 Data</button>

  <!-- Render Graph1 component with the selected file name -->
  {#if selectedFileName}
    <Graph1 fileName={selectedFileName} />
  {/if}
</div>
