<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';
  
    export let data;
    export let type;
  
    let chartData = [];
  
    // Function to create the stacked bar chart
    function createBarChart() {
      // Clear previous chart
      d3.select('.chart-container svg').remove();
  
      // Define chart dimensions
      const width = window.innerWidth * 0.8;
      const height = 700;
      const margin = { top: 70, right: 70, bottom: 200, left: 120 };
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;
  
      // Create SVG element
      const svg = d3.select('.chart-container')
                    .append('svg')
                    .attr('width', width)
                    .attr('height', height)
                    .append('g')
                    .attr('transform', `translate(${margin.left},${margin.top})`);
  
      // Define scales
      const xScale = d3.scaleBand()
                       .domain(chartData.map(d => d.team))
                       .range([0, innerWidth])
                       .padding(0.1);
  
      const yScale = d3.scaleLinear()
                       .domain([0, 8000])
                       .nice()
                       .range([innerHeight, 0]);
  
      if (type == "Total Yards") {
         // Create and append bars for passing yards
         svg.selectAll('.passing-bar')
            .data(chartData)
            .enter()
            .append('rect')
            .attr('class', 'passing-bar')
            .attr('x', d => xScale(d.team))
            .attr('y', d => yScale(d.pass_yds + d.rush_yds))
            .attr('width', xScale.bandwidth())
            .attr('height', d => innerHeight - yScale(d.pass_yds))
            .attr('fill', 'steelblue');

         // Create and append bars for rushing yards
         svg.selectAll('.rushing-bar')
            .data(chartData)
            .enter()
            .append('rect')
            .attr('class', 'rushing-bar')
            .attr('x', d => xScale(d.team))
            .attr('y', d => yScale(d.rush_yds))
            .attr('width', xScale.bandwidth())
            .attr('height', d => yScale(d.pass_yds) - yScale(d.pass_yds + d.rush_yds))
            .attr('fill', 'green');
      }

      if (type == "Passing Yards") {
         // Create and append bars for passing yards
         svg.selectAll('.passing-bar')
            .data(chartData)
            .enter()
            .append('rect')
            .attr('class', 'passing-bar')
            .attr('x', d => xScale(d.team))
            .attr('y', d => yScale(d.pass_yds))
            .attr('width', xScale.bandwidth())
            .attr('height', d => yScale(d.rush_yds) - yScale(d.pass_yds + d.rush_yds))
            .attr('fill', 'steelblue');
      }
      
      if (type == "Rushing Yards") {
         // Create and append bars for rushing yards
         svg.selectAll('.rushing-bar')
            .data(chartData)
            .enter()
            .append('rect')
            .attr('class', 'rushing-bar')
            .attr('x', d => xScale(d.team))
            .attr('y', d => yScale(d.rush_yds))
            .attr('width', xScale.bandwidth())
            .attr('height', d => yScale(d.pass_yds) - yScale(d.pass_yds + d.rush_yds))
            .attr('fill', 'green');
      }
  
      // Create X axis
      svg.append('g')
         .attr('class', 'x-axis')
         .attr('transform', `translate(0,${innerHeight})`)
         .call(d3.axisBottom(xScale))
         .selectAll('text')
         .attr('transform', 'rotate(-45)')
         .style('text-anchor', 'end')
         .attr('font-size', '16px');
  
      // Create Y axis
      svg.append('g')
         .attr('class', 'y-axis')
         .call(d3.axisLeft(yScale).ticks(10))
         .selectAll('text')
         .attr('font-size', '16px');
  
      // Add X axis label
      svg.append('text')
         .attr('class', 'x-axis-label')
         .attr('x', innerWidth / 2)
         .attr('y', innerHeight + margin.bottom * 1)
         .attr('text-anchor', 'middle')
         .text('NFL Teams')
         .attr('font-size', '20px');
  
      // Add Y axis label
      svg.append('text')
         .attr('class', 'y-axis-label')
         .attr('transform', 'rotate(-90)')
         .attr('x', -innerHeight / 2)
         .attr('y', -margin.left / 2)
         .attr('text-anchor', 'middle')
         .text('Yards')
         .attr('font-size', '20px');
    }
    
      

    // Call the createBarChart function when the component mounts
    onMount(() => {
      createBarChart();
    });
  
    // Update chart data when data prop changes
    afterUpdate(() => {
      chartData = data;
      createBarChart();
    });
  </script>
  
  <div class="chart-container"></div>
  
  <style>
    /* Add your CSS styles here */
  </style>