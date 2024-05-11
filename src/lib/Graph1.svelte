<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import { scaleLinear, scalePoint } from 'd3-scale';
  import { line, area } from 'd3-shape';

  export let fileName; // Dynamic CSV file name as a prop

  let csvData = []; // Holds the parsed CSV data

  // Load the CSV data
  onMount(async () => {
    try {
      csvData = await csv(`graph1files/${fileName}.csv`); // Load CSV data dynamically from Graph1 folder
      // Convert the 'Sales_count' and 'Inventory_count' to numbers
      csvData.forEach((d) => {
        d.Sales_count = +d.Sales_count; // Ensure 'Sales_count' is a number
        d.Inventory_count = +d.Inventory_count; // Ensure 'Inventory_count' is a number
      });
    } catch (error) {
      console.error('Error loading CSV:', error);
    }
  });

  // Define scales for the x-axis and y-axis
  let xScale = scalePoint().range([0, 400]); // X-axis scale
  let yScale = scaleLinear().range([200, 0]); // Y-axis scale

  // Update scales on data load
  $: {
    if (csvData.length > 0) {
      xScale.domain(csvData.map(d => d.month)).padding(0.5);
      yScale.domain([0, Math.max(...csvData.map(d => d.Sales_count, d => d.Inventory_count))]);
    }
  }

  // Define line and area generators for sales and inventory
  let salesLineGenerator = line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.Sales_count));

  let inventoryAreaGenerator = area()
    .x(d => xScale(d.month))
    .y0(yScale(0))
    .y1(d => yScale(d.Inventory_count));
</script>

<!-- Graph title -->
<h2>Forecasted Amount of Orders vs. Actual Amount of Orders</h2>

<style>
  .chart {
    position: relative;
    width: 500px;
    height: 300px;
    border: 1px solid black;
    margin: 20px;
    padding: 20px;
  }

  .line {
    fill: none;
    stroke-width: 2;
  }

  .sales-line {
    stroke: blue;
  }

  .inventory-area {
    fill: red;
    opacity: 0.5;
  }

  .legend {
    position: absolute;
    top: 10px;
    right: 10px;
  }

  .legend-item {
    display: flex;
    align-items: center;
    margin-bottom: 5px;
  }

  .legend-item .color-box {
    width: 10px;
    height: 10px;
    margin-right: 5px;
  }

  .y-axis {
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 20px 0;
    margin-left: -40px; /* Adjust for axis labels */
  }

  .axis-text {
    font-size: 10px;
    text-anchor: middle;
  }
</style>

<div class="chart">
  <!-- Legend -->
  <div class="legend" style="top: 10px; right: 10px;">
    <div class="legend-item">
      <div class="color-box" style="background-color: blue;"></div>
      <span>Sales</span>
    </div>
    <div class="legend-item">
      <div class="color-box" style="background-color: red;"></div>
      <span>Inventory</span>
    </div>
  </div>

  <!-- Draw lines for sales and inventory -->
  {#if csvData.length > 0}
    <svg width="100%" height="100%">
      <!-- X-axis -->
      <g transform="translate(0, 200)">
        <line x1="0" y1="0" x2="400" y2="0" stroke="black" />
        {#each xScale.domain() as month, index}
          <text class="axis-text" x={xScale(month)} y="20">{month}</text>
        {/each}
      </g>
      <!-- Y-axis -->
      <!-- Removed Y-axis vertical line -->
      {#each yScale.ticks(5) as tick}
        <text class="axis-text" x="-20" y={yScale(tick)}>{tick}</text>
      {/each}
      <!-- Sales line -->
      <path class="line sales-line" d={salesLineGenerator(csvData)} />
      <!-- Inventory area -->
      <path class="inventory-area" d={inventoryAreaGenerator(csvData)} />
    </svg>
  {/if}
</div>
