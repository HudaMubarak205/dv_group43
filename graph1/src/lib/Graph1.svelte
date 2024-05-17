<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import { scaleLinear, scalePoint } from 'd3-scale';
  import { line, area } from 'd3-shape';

  export let fileName; 

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
  let xScale = scalePoint().range([50, 400]); // X-axis scale
  let yScale = scaleLinear().range([200, 20]); // Y-axis scale

  // Update scales on data load
  $: {
    if (csvData.length > 0) {
      xScale.domain(csvData.map(d => d.month)).padding(0.5);
      yScale.domain([0, Math.max(...csvData.map(d => d.Sales_count, d => d.Inventory_count))]);
    }
  }

  // Define line and area generators for forecasted and actual number of orders
  let salesLineGenerator = line()
    .x(d => xScale(d.month))
    .y(d => yScale(d.Sales_count));

  let inventoryAreaGenerator = area()
    .x(d => xScale(d.month))
    .y0(yScale(0))
    .y1(d => yScale(d.Inventory_count));

  let hoveredItem = null;

  function handleCircleHover(item) {
    hoveredItem = item;
  }
</script>

<style>
  .chart {
    position: relative;
    width: 800px; /* Increased width */
    height: 260px;
    /* border: 1px solid black; */
    margin: 40px auto; /* Keep the border and center the chart */
    padding: 40px;
    margin-left: 50px; /* Move the chart to the right */
  }

  .line {
    fill: none;
    stroke-width: 2;
  }

  .sales-line {
    stroke: blue;
  }

  .inventory-area {
    fill: green;
    opacity: 0.5;
  }

  .legend {
    position: absolute;
    bottom: 100px; /* Move to the bottom */
    right: 150px; /* Adjusted right positioning */
  }

  .legend-item {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
  }

  .legend-item .color-box {
    width: 10px;
    height: 10px;
    margin-right: 5px;
  }

  .legend-item .color-circle {
    width: 10px;
    height: 10px;
    border-radius: 50%; /* Make it a circle */
    margin-right: 5px;
  }

  .y-axis {
    font-size: "10px";
    font-family: sans-serif;
    text-anchor: "end";
  }

  .axis-text {
    font-size: 10px;
    text-anchor: middle;
  }

</style>

<div class="chart">
  <!-- Legend -->
  <div class="legend">
    <div class="legend-item">
      <div class="color-box" style="background-color: blue;"></div>
      <span>Forecasted number of orders</span> <!-- Two lines for the first item -->
    </div>
    <div class="legend-item">
      <div class="color-box" style="background-color: rgba(0, 128, 0, 0.5);"></div> <!-- Updated with red color and opacity -->
      <span>Actual number of orders</span> <!-- Two lines for the first item -->
    </div>
    <div class="legend-item">
      <div class="color-circle" style="background-color: green;"></div>
      <span>Overstock</span>
    </div>
    <div class="legend-item">
      <div class="color-circle" style="background-color: red;"></div>
      <span>Out of stock</span>
    </div>
  </div>

  <!-- Draw lines for sales forecasted and actual nymber of orders -->
  {#if csvData.length > 0}
    <svg width="100%" height="100%">
      <!-- X-axis -->
      <g transform="translate(0, 220)">
        <line x1="50" y1="-20" x2="400" y2="-20" stroke="black" />
        {#each xScale.domain() as month, index}
          <text class="axis-text" x={xScale(month)} y="0">{month}</text>
        {/each}
      </g>
      <!-- Y-axis -->
      <g class="y-axis">
        <!-- Y-axis line -->
        <line x1="50" y1="0" x2="50" y2="200" stroke="black" />
        <!-- Y-axis ticks and labels -->
        {#each yScale.ticks(5) as tick}
          <g transform={`translate(0, ${yScale(tick)})`}>
            <!-- Adjusted text positioning to the left of the y-axis line -->
            <text class="axis-text" x="15" y="3">{tick}</text>
            <!-- Adjusted tick line to extend to the left -->
            <line x1="-5" y1="0" x2="0" y2="0" stroke="black" />
          </g>
        {/each}
      </g>
      <!-- Forcasted orders line -->
      <path class="line sales-line" d={salesLineGenerator(csvData)} />
      <!-- Actual orders area -->
      <path class="inventory-area" d={inventoryAreaGenerator(csvData)} />

      <!-- Circle elements for hover -->
      {#each csvData as item}
      <g>
        {#if item.Inventory_count - item.Sales_count >= 4500}
          <circle cx={xScale(item.month)} cy={yScale(item.Sales_count)} r="7" fill="red"
            on:mouseover={() => handleCircleHover(item)} 
            on:mouseout={() => handleCircleHover(null)} 
            role="img" aria-label={`Inventory count: ${item.Inventory_count}, Sales count: ${item.Sales_count}`}
            style="transition: fill 0.3s ease;"
            style:fill={hoveredItem === item ? 'yellow' : 'red'} />
        {:else if item.Inventory_count - item.Sales_count <= -4500}
          <circle cx={xScale(item.month)} cy={yScale(item.Sales_count)} r="7" fill="green"
            on:mouseover={() => handleCircleHover(item)} 
            on:mouseout={() => handleCircleHover(null)} 
            role="img" aria-label={`Inventory count: ${item.Inventory_count}, Sales count: ${item.Sales_count}`}
            style="transition: fill 0.3s ease;"
            style:fill={hoveredItem === item ? 'yellow' : 'green'} />
        {/if}
      </g>
      {/each}
      
      <!-- Tooltip -->
      {#if hoveredItem !== null}
      <g>
        <rect x={xScale(hoveredItem.month) + 5} y={yScale(hoveredItem.Sales_count) - 20} width="250" height="20" fill="lightgrey" />
        <text x={xScale(hoveredItem.month) + 10} y={yScale(hoveredItem.Sales_count) - 10} font-size="12">{`Actual orders: ${hoveredItem.Inventory_count}, Forecasted orders: ${hoveredItem.Sales_count}`}</text>
      </g>
      {/if}
    </svg>
  {/if}
</div>
