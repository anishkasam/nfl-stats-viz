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
  const yardTypes = ["Total Yards", "Passing Yards", "Rushing Yards"];
  let dataForYear = data.filter(entry => entry.year === selectedYear);

  // Function to update chart data when year changes
  function updateChartData(year, type) {
    selectedYear = year;
    selectedType = type;
    dataForYear = data.filter(entry => entry.year === year);
    dispatch('updateChartData', dataForYear);
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
    <select bind:value={selectedYear} on:change={() => updateChartData(selectedYear, selectedType)}>
      {#each years as year}
        <option value={year}>{year}</option>
      {/each}
    </select>
    <select bind:value={selectedType} on:change={() => updateChartData(selectedYear, selectedType)}>
      {#each yardTypes as type}
        <option value={type}>{type}</option>
      {/each}
    </select>
    <div>
      {#if playing}
        <button on:click={togglePlay}>Stop</button>
      {:else}
        <button on:click={togglePlay}>Play</button>
      {/if}
    </div>
    <BarChart data={dataForYear} type={selectedType}/>
  </div>
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
  h1 {
    font-family: "Inter";
    font-weight: 400;
  }

  select {
    font-family: "Inter";
    font-size: 16px;
    padding: 8px;
    border-radius: 5px;
    border: 1px solid #ccc;
    margin-right: 10px;
  }

  .container {
    margin-left: 100px;
    margin-right: 100px;
    text-align: center;
  }

  button {
    font-family: "Inter";
    font-size: 16px;
    padding: 8px 16px;
    border-radius: 5px;
    border: none;
    background-color: #007bff;
    color: #fff;
    cursor: pointer;
    margin: 10px;
  }
</style>
