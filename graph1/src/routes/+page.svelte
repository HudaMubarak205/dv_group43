<script>
  import { writable } from 'svelte/store';
  import Graph1 from '$lib/Graph1.svelte'; // Import the Graph1 component
  import { setContext } from 'svelte'



  // Initialize the selected file name as a writable store
  let selectedFileName = writable('Forcast_ActualSales2024');

// Function to handle button click and set the selected file name
function selectFileName(fileName) {
  selectedFileName.set(fileName); // Set the value of the writable store
}
</script>

<!-- Graph title -->
<h2>Forecasted Amount of Orders vs. Actual Amount of Orders</h2>

<style>
  .glow-button {
    background-color: lightblue;
    border: none;
    color: darkblue; /* Text color changed to dark blue */
    padding: 10px 20px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 8px;
    transition-duration: 0.4s;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
  }

  .glow-button.selected {
    background-color: lightgrey; /* Change color to light grey when selected */
  }

  .glow-button:hover {
    background-color: lightblue;
    box-shadow: 0 12px 16px 0 rgba(0, 0, 0, 0.24), 0 17px 50px 0 rgba(0, 0, 0, 0.19);
  }
</style>

<div>
  <!-- Button to select "Forcast_ActualSales2022" file name -->
  <button class="glow-button {($selectedFileName === 'Forcast_ActualSales2022') ? 'selected' : ''}" on:click={() => selectFileName("Forcast_ActualSales2022")}>Load 2022 Data</button>
  <!-- Button to select "Forcast_ActualSales2023" file name -->
  <button class="glow-button {($selectedFileName === 'Forcast_ActualSales2023') ? 'selected' : ''}" on:click={() => selectFileName("Forcast_ActualSales2023")}>Load 2023 Data</button>
  <!-- Button to select "Forcast_ActualSales2024" file name -->
  <button class="glow-button {($selectedFileName === 'Forcast_ActualSales2024') ? 'selected' : ''}" on:click={() => selectFileName("Forcast_ActualSales2024")}>Load 2024 Data</button>


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
