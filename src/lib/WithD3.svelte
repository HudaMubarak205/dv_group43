<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import { scaleLinear, scalePoint } from 'd3-scale';
  import { line } from 'd3-shape';

  let csvData = []; // Holds the parsed CSV data

  // Load the CSV data from the `static` directory
  onMount(async () => {
    csvData = await csv('/sales_inventory.csv'); // Load CSV data

    // Convert the 'Sales_count' and 'Inventory_count' to numbers
    csvData.forEach((d) => {
      d.Sales_count = +d.Sales_count; // Ensure 'Sales_count' is a number
      d.Inventory_count = +d.Inventory_count; // Ensure 'Inventory_count' is a number
    });
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

  // Define line generators for sales and inventory
  let salesLineGenerator = line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.Sales_count));

  let inventoryLineGenerator = line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.Inventory_count));
</script>

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

  .inventory-line {
    stroke: red;
  }

  .axis-text {
    font-size: 10px;
    text-anchor: middle;
  }

  .legend {
    display: flex;
    justify-content: center;
    margin-top: 10px;
  }

  .legend-item {
    display: flex;
    align-items: center;
    margin-right: 20px;
  }

  .legend-item .color-box {
    width: 10px;
    height: 10px;
    margin-right: 5px;
  }
</style>

<div class="chart">
  <!-- Draw lines for sales and inventory -->
  {#if csvData.length > 0}
    <svg width="100%" height="100%">
      <path class="line sales-line" d={salesLineGenerator(csvData)} />
      <path class="line inventory-line" d={inventoryLineGenerator(csvData)} />
      <!-- X-axis -->
      <g transform="translate(0, 200)">
        <line x1="0" y1="0" x2="400" y2="0" stroke="black" />
        {#each xScale.domain() as month, index}
          <text class="axis-text" x={xScale(month)} y="15">{month}</text>
        {/each}
      </g>
      <!-- Y-axis -->
      <g transform="translate(0, 0)">
        <line x1="0" y1="0" x2="0" y2="200" stroke="black" />
        {#each yScale.ticks(5) as tick}
          <text class="axis-text" x="-20" y={yScale(tick)}>{tick}</text>
        {/each}
      </g>
    </svg>
  {/if}
  <!-- Legend -->
  <div class="legend">
    <div class="legend-item">
      <div class="color-box" style="background-color: blue;"></div>
      <span>Sales</span>
    </div>
    <div class="legend-item">
      <div class="color-box" style="background-color: red;"></div>
      <span>Inventory</span>
    </div>
  </div>
</div>
