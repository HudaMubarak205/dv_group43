<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3'; // D3 for data processing and visualization
  
    let csvData = []; // Holds the parsed CSV data
  
    // Load the CSV data from the `static` directory
    onMount(async () => {
      csvData = await d3.csv('/sales_inventory.csv'); // Load CSV data
  
      // Convert the 'Sales_count' and 'Inventory_count' to numbers
      csvData.forEach((d) => {
        d.Sales_count = Number(d.Sales_count); // Ensure 'Sales_count' is a number
        d.Inventory_count = Number(d.Inventory_count); // Ensure 'Inventory_count' is a number
      });
    });
  
    // Define scales for the x-axis and y-axis
    const xScale = d3.scaleBand()
      .domain(csvData.map((d) => d.month)) // Use 'month' for x-axis
      .range([0, 400])
      .padding(0.1); // Padding for spacing
  
    const yScale = d3.scaleLinear()
      .domain([0, d3.max(csvData, (d) => Math.max(d.Sales_count, d.Inventory_count))])
      .range([200, 0]); // Flip because SVG starts from the top
  
    // Define line generators for sales and inventory
    const salesLineGenerator = d3.line()
      .x((d) => xScale(d.month) + xScale.bandwidth() / 2)
      .y((d) => yScale(d.Sales_count)); // Sales line
  
    const inventoryLineGenerator = d3.line()
      .x((d) => xScale(d.month) + xScale.bandwidth() / 2)
      .y((d) => yScale(d.Inventory_count)); // Inventory line
  </script>
  
  <svg width="500" height="300">
    <!-- X-axis -->
    <g transform="translate(50, 200)">
      <line x1="0" y1="0" x2="400" y2="0" stroke="black" />
  
      {#each xScale.domain() as month}
        <text
          x="{xScale(month) + xScale.bandwidth() / 2}"
          y="20"
          text-anchor="middle"
        >
          {month}
        </text>
      {/each}
    </g>
  
    <!-- Y-axis -->
    <g transform="translate(50, 0)">
      <line x1="0" y1="0" x2="0" y2="200" stroke="black" />
  
      {#each d3.range(0, yScale.domain()[1] + 1, 50) as y}
        <text
          x="-10"
          y="{yScale(y)}"
          text-anchor="end"
        >
          {y}
        </text>
      {/each}
    </g>
  
    <!-- Draw lines for sales and inventory -->
    <g transform="translate(50, 0)">
      {#if csvData.length > 0}
        <path
          d="{salesLineGenerator(csvData)}"
          stroke="blue"
          fill="none"
          stroke-width="2"
        />
  
        <path
          d="{inventoryLineGenerator(csvData)}"
          stroke="red"
          fill="none"
          stroke-width="2"
        />
      {/if}
    </g>
  </svg>
  