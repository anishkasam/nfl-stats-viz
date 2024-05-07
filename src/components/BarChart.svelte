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
      const height = window.innerHeight * 0.8;
      const margin = { top: 70, right: 100, bottom: 200, left: 100};
      const innerWidth = width - 100;
      const innerHeight = height - margin.top - margin.bottom;
      const rushColor = '#eb4d4b';
      const passColor = '#22a6b3';
  
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
            .attr('fill', passColor);

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
            .attr('fill', rushColor);
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
            .attr('fill', passColor);
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
            .attr('fill', rushColor);
      }
  
      // Create X axis
      svg.append('g')
         .attr('class', 'x-axis')
         .attr('transform', `translate(0,${innerHeight})`)
         .call(d3.axisBottom(xScale))
         .selectAll('text')
         .attr('transform', 'rotate(-45)')
         .style('text-anchor', 'end')
         .attr('font-family', 'Inter')
         .attr('font-size', '16px');
  
      // Create Y axis
      svg.append('g')
         .attr('class', 'y-axis')
         .call(d3.axisLeft(yScale).ticks(10))
         .selectAll('text')
         .attr('font-family', 'Inter')
         .attr('font-size', '16px');
  
      // Add X axis label
      svg.append('text')
         .attr('class', 'x-axis-label')
         .attr('x', innerWidth / 2)
         .attr('y', innerHeight + margin.bottom * 0.9)
         .attr('text-anchor', 'middle')
         .text('Team Name')
         .attr('font-family', 'Inter')
         .attr('font-size', '24px');
  
      // Add Y axis label
      svg.append('text')
         .attr('class', 'y-axis-label')
         .attr('transform', 'rotate(-90)')
         .attr('x', -innerHeight / 2)
         .attr('y', -margin.left / 1.2)
         .attr('text-anchor', 'middle')
         .text('Yards')
         .attr('font-family', 'Inter')
         .attr('font-size', '24px');
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
      .svg-container {
         display: inline-block;
         position: relative;
         width: 100%;
         padding-bottom: 100%; /* aspect ratio */
         vertical-align: top;
         overflow: hidden;
      }

      .svg-content-responsive {
         display: inline-block;
         position: absolute;
         top: 10px;
         left: 0;
      }
   </style>