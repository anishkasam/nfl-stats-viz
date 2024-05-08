<script>
  import { onMount, afterUpdate } from 'svelte';
  import * as d3 from 'd3';

  export let data;
  export let type;

  let svg, xScale, yScale;
  const margin = { top: 70, right: 100, bottom: 200, left: 100};
  let width, height, innerWidth, innerHeight;
  const rushColor = '#eb4d4b';
  const passColor = '#22a6b3';

  function initChart() {
    // Define chart dimensions
    width = window.innerWidth * 0.8;
    height = window.innerHeight * 0.8;
    innerWidth = width - 100;
    innerHeight = height - margin.top - margin.bottom;

    // Create SVG element
    svg = d3.select('.chart-container')
            .append('svg')
            .attr('width', width)
            .attr('height', height)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

    // Define scales
    xScale = d3.scaleBand()
               .range([0, innerWidth])
               .padding(0.1);
    yScale = d3.scaleLinear()
               .range([innerHeight, 0]);

    // Create X and Y axes
    svg.append('g')
       .attr('class', 'x-axis')
       .attr('transform', `translate(0,${innerHeight})`);

    svg.append('g')
       .attr('class', 'y-axis');

    // Add axis labels
    svg.append('text')
       .attr('class', 'x-axis-label')
       .attr('x', innerWidth / 2)
       .attr('y', innerHeight + margin.bottom * 0.9)
       .attr('text-anchor', 'middle')
       .text('Team Name')
       .attr('font-family', 'Inter')
       .attr('font-size', '24px');

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

   let previousType = "";

function updateChart(time) {
   {
   // Update scales
   xScale.domain(data.map(d => d.team));
   yScale.domain([0, 8000]); // Ensure scale is correct for total yards

   // Update the axes
   svg.select('.x-axis')
      .call(d3.axisBottom(xScale))
      .selectAll('text')
      .style('text-anchor', 'end')
      .attr('dx', '-.8em')
      .attr('dy', '.15em')
      .attr('transform', 'rotate(-45)').attr('font-family', 'Inter')
         .attr('font-size', '14px');
   }

   svg.select('.y-axis').call(d3.axisLeft(yScale)).attr('font-family', 'Inter')
         .attr('font-size', '14px');

   if (previousType == "" && type == "Total Yards") {
      // Update rushing bars depending on the selection
      const rushBars = svg.selectAll('.rushing-bar').data(type === 'Rushing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      rushBars.enter()
         .append('rect')
         .attr('class', 'rushing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', d => yScale(d.rush_yds + d.pass_yds)) // Start on top of passing bars
         .attr('width', xScale.bandwidth())
         .attr('height', d => innerHeight - yScale(d.rush_yds))
         .attr('fill', rushColor)
         .merge(rushBars)
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.rush_yds + d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.rush_yds));

      rushBars.exit()
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds)) // Animate to the top of the passing bars
         .attr('height', 0)
         .remove();

      // Update passing bars depending on the selection
      const passBars = svg.selectAll('.passing-bar').data(type === 'Passing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      passBars.enter()
         .append('rect')
         .attr('class', 'passing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', innerHeight) // Start from the bottom
         .attr('width', xScale.bandwidth())
         .attr('height', 0) // Start with zero height
         .attr('fill', passColor)
         .merge(passBars) // Merge enter and update selections
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));

      passBars.exit()
         .transition()
         .duration(time)
         .attr('y', innerHeight) // Animate to the bottom
         .attr('height', 0)
         .remove();
   }

   else if (previousType == "Total Yards" && type == "Passing Yards") {
      console.log("in here")
      // Update rushing bars depending on the selection
      const rushBars = svg.selectAll('.rushing-bar').data(type === 'Rushing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      rushBars.enter()
         .append('rect')
         .attr('class', 'rushing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', d => yScale(d.rush_yds + d.pass_yds)) // Start on top of passing bars
         .attr('width', xScale.bandwidth())
         .attr('height', d => innerHeight - yScale(d.rush_yds))
         .attr('fill', rushColor)
         .merge(rushBars)
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.rush_yds + d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.rush_yds));

      rushBars.exit()
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds)) // Animate to the top of the passing bars
         .attr('height', 0)
         .remove();

      // Update passing bars depending on the selection
      const passBars = svg.selectAll('.passing-bar').data(type === 'Passing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      passBars.enter()
         .append('rect')
         .attr('class', 'passing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', innerHeight) // Start from the bottom
         .attr('width', xScale.bandwidth())
         .attr('height', 0) // Start with zero height
         .attr('fill', passColor)
         .merge(passBars) // Merge enter and update selections
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));

      passBars.exit()
         .transition()
         .duration(time)
         .attr('y', innerHeight) // Animate to the bottom
         .attr('height', 0)
         .remove();
   }

   else if (previousType == "Passing Yards" && type == "Total Yards") {
      // Update rushing bars depending on the selection
      const rushBars = svg.selectAll('.rushing-bar').data(type === 'Rushing Yards' || type === 'Total Yards' ? data : [], d => d.team);
        rushBars.enter()
            .append('rect')
            .attr('class', 'rushing-bar')
            .attr('x', d => xScale(d.team))
            .attr('width', xScale.bandwidth())
            .attr('y', d => yScale(d.pass_yds)) // Start at the top of the passing bars
            .attr('height', 0) // Start with zero height
            .attr('fill', rushColor)
          .merge(rushBars)
            .transition()
            .duration(time)
            .attr('y', d => yScale(d.rush_yds + d.pass_yds)) // Move up to show total yards
            .attr('height', d => innerHeight - yScale(d.rush_yds));

        rushBars.exit()
            .transition()
            .duration(time)
            .attr('y', innerHeight)
            .attr('height', 0)
            .remove();

      // Update passing bars depending on the selection
      const passBars = svg.selectAll('.passing-bar').data(type === 'Passing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      passBars.enter()
         .append('rect')
         .attr('class', 'passing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', innerHeight) // Start from the bottom
         .attr('width', xScale.bandwidth())
         .attr('height', 0) // Start with zero height
         .attr('fill', passColor)
         .merge(passBars) // Merge enter and update selections
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));

      passBars.exit()
         .transition()
         .duration(time)
         .attr('y', innerHeight) // Animate to the bottom
         .attr('height', 0)
         .remove();
   }

   else if (previousType == "Passing Yards" && type == "Rushing Yards") {
      // Update rushing bars depending on the selection
      const rushBars = svg.selectAll('.rushing-bar').data(type === 'Rushing Yards' || type === 'Total Yards' ? data : [], d => d.team);
        rushBars.enter()
            .append('rect')
            .attr('class', 'rushing-bar')
            .attr('x', d => xScale(d.team))
            .attr('width', xScale.bandwidth())
            .attr('y', innerHeight) // Start at the top of the passing bars
            .attr('height', 0) // Start with zero height
            .attr('fill', rushColor)
          .merge(rushBars)
            .transition()
            .duration(time)
            .attr('y', d => yScale(d.rush_yds)) // Move up to show total yards
            .attr('height', d => innerHeight - yScale(d.rush_yds));

        rushBars.exit()
            .transition()
            .duration(time)
            .attr('y', innerHeight)
            .attr('height', 0)
            .remove();

      // Update passing bars depending on the selection
      const passBars = svg.selectAll('.passing-bar').data(type === 'Passing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      passBars.enter()
         .append('rect')
         .attr('class', 'passing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', innerHeight) // Start from the bottom
         .attr('width', xScale.bandwidth())
         .attr('height', 0) // Start with zero height
         .attr('fill', passColor)
         .merge(passBars) // Merge enter and update selections
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));

      passBars.exit()
         .transition()
         .duration(time)
         .attr('y', innerHeight) // Animate to the bottom
         .attr('height', 0)
         .remove();
   }

   else if (previousType == "Rushing Yards" && type == "Passing Yards") {
      // Update rushing bars depending on the selection
      const rushBars = svg.selectAll('.rushing-bar').data(type === 'Rushing Yards' || type === 'Total Yards' ? data : [], d => d.team);
        rushBars.enter()
            .append('rect')
            .attr('class', 'rushing-bar')
            .attr('x', d => xScale(d.team))
            .attr('width', xScale.bandwidth())
            .attr('y', innerHeight) // Start at the top of the passing bars
            .attr('height', 0) // Start with zero height
            .attr('fill', rushColor)
          .merge(rushBars)
            .transition()
            .duration(time)
            .attr('y', d => yScale(d.rush_yds)) // Move up to show total yards
            .attr('height', d => innerHeight - yScale(d.rush_yds));

        rushBars.exit()
            .transition()
            .duration(time)
            .attr('y', innerHeight)
            .attr('height', 0)
            .remove();

      // Update passing bars depending on the selection
      const passBars = svg.selectAll('.passing-bar').data(type === 'Passing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      passBars.enter()
         .append('rect')
         .attr('class', 'passing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', innerHeight) // Start from the bottom
         .attr('width', xScale.bandwidth())
         .attr('height', 0) // Start with zero height
         .attr('fill', passColor)
         .merge(passBars) // Merge enter and update selections
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));

      passBars.exit()
         .transition()
         .duration(time)
         .attr('y', innerHeight) // Animate to the bottom
         .attr('height', 0)
         .remove();
   }

   else if (previousType == "Total Yards" && type == "Rushing Yards") {
      // Update rushing bars depending on the selection
      const rushBars = svg.selectAll('.rushing-bar').data(type === 'Rushing Yards' || type === 'Total Yards' ? data : [], d => d.team);
        rushBars.enter()
            .append('rect')
            .attr('class', 'rushing-bar')
            .attr('x', d => xScale(d.team))
            .attr('width', xScale.bandwidth())
            .attr('y', innerHeight) // Start at the top of the passing bars
            .attr('height', 0) // Start with zero height
            .attr('fill', rushColor)
          .merge(rushBars)
            .transition()
            .duration(time)
            .attr('y', d => yScale(d.rush_yds)) // Move up to show total yards
            .attr('height', d => innerHeight - yScale(d.rush_yds));

        rushBars.exit()
            .transition()
            .duration(time)
            .attr('y', innerHeight)
            .attr('height', 0)
            .remove();

      // Update passing bars depending on the selection
      const passBars = svg.selectAll('.passing-bar').data(type === 'Passing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      passBars.enter()
         .append('rect')
         .attr('class', 'passing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', innerHeight) // Start from the bottom
         .attr('width', xScale.bandwidth())
         .attr('height', 0) // Start with zero height
         .attr('fill', passColor)
         .merge(passBars) // Merge enter and update selections
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));

      passBars.exit()
         .transition()
         .duration(time)
         .attr('y', innerHeight) // Animate to the bottom
         .attr('height', 0)
         .remove();
   }

   else if (previousType == "Rushing Yards" && type == "Total Yards") {
      // Update rushing bars depending on the selection
      const rushBars = svg.selectAll('.rushing-bar').data(type === 'Rushing Yards' || type === 'Total Yards' ? data : [], d => d.team);
        rushBars.enter()
            .append('rect')
            .attr('class', 'rushing-bar')
            .attr('x', d => xScale(d.team))
            .attr('width', xScale.bandwidth())
            .attr('y', innerHeight) // Start at the top of the passing bars
            .attr('height', 0) // Start with zero height
            .attr('fill', rushColor)
          .merge(rushBars)
            .transition()
            .duration(time)
            .attr('y', d => yScale(d.rush_yds + d.pass_yds)) // Move up to show total yards
            .attr('height', d => innerHeight - yScale(d.rush_yds));

        rushBars.exit()
            .transition()
            .duration(time)
            .attr('y', innerHeight)
            .attr('height', 0)
            .remove();

      // Update passing bars depending on the selection
      const passBars = svg.selectAll('.passing-bar').data(type === 'Passing Yards' || type === 'Total Yards' ? data : [], d => d.team);

      passBars.enter()
         .append('rect')
         .attr('class', 'passing-bar')
         .attr('x', d => xScale(d.team))
         .attr('y', innerHeight) // Start from the bottom
         .attr('width', xScale.bandwidth())
         .attr('height', 0) // Start with zero height
         .attr('fill', passColor)
         .merge(passBars) // Merge enter and update selections
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));

      passBars.exit()
         .transition()
         .duration(time)
         .attr('y', innerHeight) // Animate to the bottom
         .attr('height', 0)
         .remove();
   }

   else if (previousType == "Total Yards" && type == "Total Yards") {
      // Directly update rushing and passing bars if they already exist
      svg.selectAll('.rushing-bar')
         .data(data, d => d.team)
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.rush_yds + d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.rush_yds));

      svg.selectAll('.passing-bar')
         .data(data, d => d.team)
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));
   }

   else if (previousType == "Rushing Yards" && type == "Rushing Yards") {
      // Only update rushing bars
      svg.selectAll('.rushing-bar')
         .data(data, d => d.team)
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.rush_yds))
         .attr('height', d => innerHeight - yScale(d.rush_yds));
   }

   else if (previousType == "Passing Yards" && type == "Passing Yards") {
      // Only update passing bars
      svg.selectAll('.passing-bar')
         .data(data, d => d.team)
         .transition()
         .duration(time)
         .attr('y', d => yScale(d.pass_yds))
         .attr('height', d => innerHeight - yScale(d.pass_yds));
   }

   previousType = type;
}

   

  onMount(() => {
    initChart();
    updateChart(0);
  });

  afterUpdate(() => {
    updateChart(500);
  });
</script>

<div class="chart-container"></div>
