<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import data from '../data/data.json';
  import BarChart from './BarChart.svelte';

  // Dispatch event to pass selected year to BarChart component
  const dispatch = createEventDispatcher();

  // Filter unique years from data
  const years = [...new Set(data.map(entry => entry.year))];

  // Set initial year
  let selectedYear = 2003;
  let selectedType = "Total Yards";
  let lowercaseType = "total yards";
  const yardTypes = ["Total Yards", "Passing Yards", "Rushing Yards"];

  let dataForYear = data.filter(entry => entry.year === selectedYear);
  dataForYear.sort((a, b) => a.team.localeCompare(b.team));

  let average = dataForYear.reduce((acc, cur) => acc + cur.rush_yds + cur.pass_yds, 0) / 32;

  // Function to update chart data when year changes
  function updateChartData(year, type) {
    selectedYear = year;
    selectedType = type;
    lowercaseType = selectedType.toLowerCase();

    // sort teams alphabetically so order is preserved
    dataForYear = data.filter(entry => entry.year === year);
    dataForYear.sort((a, b) => a.team.localeCompare(b.team));

    if (selectedType == "Total Yards") {
      average = dataForYear.reduce((acc, cur) => acc + cur.rush_yds + cur.pass_yds, 0) / 32;
    }
    else if (selectedType == "Passing Yards") {
      average = dataForYear.reduce((acc, cur) => acc + cur.pass_yds, 0) / 32;
    }
    else {
      average = dataForYear.reduce((acc, cur) => acc + cur.rush_yds, 0) / 32;
    }

    dispatch('updateChartData', dataForYear);
    console.log(dataForYear);
  }

  let intervalId;
  let playing = false;

  function togglePlay() {
    if (playing) {
      clearInterval(intervalId);
      playing = false;
    } else {
      intervalId = setInterval(() => {
        if (selectedYear < Math.max(...years)) {
          selectedYear++;
          updateChartData(selectedYear, selectedType);
        } else {
          clearInterval(intervalId);
          playing = false;
        }
      }, 1000);
      playing = true;
    }
  }
</script>

<main>
  <div class="container">
    <h1>How has Passing & Rushing Changed in the NFL in the Past 20 Years?</h1>
    <div class="controls">
      <select bind:value={selectedYear} on:change={() => updateChartData(selectedYear, selectedType)}>
        {#each years as year}
          <option value={year}>{year}</option>
        {/each}
      </select>
      {#if playing}
        <button on:click={togglePlay}>Stop</button>
      {:else}
        <button on:click={togglePlay}>Play</button>
      {/if}
      <select bind:value={selectedType} on:change={() => updateChartData(selectedYear, selectedType)}>
        {#each yardTypes as type}
          <option value={type}>{type}</option>
        {/each}
      </select>
    </div>
    <h3>In {selectedYear}, each team gained an average of {Math.round(average)} {lowercaseType}</h3>
    <div class="chartContainer">
      <BarChart data={dataForYear} type={selectedType}/>
    </div>
  </div>
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
  h1 {
    font-family: "Inter";
    font-weight: 400;
  }

  h3 {
    font-family: "Inter";
    margin-top: 100px;
    font-weight: 400;
  }

  .chartContainer {
    margin-top: -125px;
  }

  select {
    font-family: "Inter";
    font-size: 16px;
    padding-top: -10px;
    border-radius: 5px;
    border: 1px solid #ccc;
  }

  .container {
    margin-left: 100px;
    margin-right: 100px;
    text-align: center;
  }

  .controls {
    margin-top: 25px;
  }

  .controls *  {
    display: inline-flex;
    font-family: "Inter";
    font-size: 16px;
    padding-top: -10px;
    border-radius: 5px;
    border: 1px solid #ccc;
    cursor: pointer;
    background: white;
    padding: 10px 15px 10px 15px;
    margin-left: 5px;
    margin-right: 5px;
    text-align: center;
  }

  .controls button {
    width: 80px;
    display: inline-block;
    text-align: center;
  }
</style>
