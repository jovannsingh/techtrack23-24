<!-- <script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import dataset from './dataset.json';
  
  export let selectedPlayer;
  
  onMount(() => {
    // Je kunt hier je code uit tweedegrafiek.svelte plaatsen en toegang krijgen tot selectedPlayer
    console.log('Geselecteerde speler:', selectedPlayer);
  });

  let el;

  onMount(() => {
    const margin = { top: 20, right: 20, bottom: 50, left: 40 };
    const width = 6200 - margin.left - margin.right; // Breedte aangepast
    const height = 100 - margin.top - margin.bottom; // Hoogte aangepast

    const color = d3.scaleOrdinal().range(['#A50044', '#004D98', '#009688']); // Kleuren voor Goals, Assists, Appearances

    const svg = d3.select(el)
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    const filteredData = dataset.filter(d => d.Player === 'Ronaldo');
    console.log(filteredData)

    const stack = d3.stack()
      .keys(['Liga_Goals', 'Liga_Asts', 'Liga_Aps'])
      .order(d3.stackOrderNone)
      .offset(d3.stackOffsetNone);

    const stackedData = stack([filteredData[0]]); // Slechts één datapunt stapelen
    console.log("stackedData", stackedData)

    var xscale  = d3.scaleLinear()
    .domain([0, 450])
    .range([0, width])
    
    svg.selectAll('.serie')
      .data(stackedData)
      .enter().append('g')
      .attr('class', 'serie')
      .attr('fill', d => color(d.key))
      .selectAll('rect')
      .data(d => d)
      .enter().append('rect')
      .attr('x', d => xscale(d[0])) // x-coördinaat voor het begin van de staaf
      .attr('y', 0) // y-coördinaat voor het begin van de staaf
      .attr('width', d => xscale(d[1]) - xscale(d[0])) // Breedte van de staaf
      .attr('height', height); // Hoogte van de staaf

    // Legenda toevoegen
    const legend = svg.selectAll('.legend')
      .data(['Liga_Goals', 'Liga_Asts', 'Liga_Aps'])
      .enter().append('g')
      .attr('class', 'legend')
      .attr('transform', (d, i) => `translate(${i * 150},${height + 30})`);

    legend.append('rect')
      .attr('width', 18)
      .attr('height', 18)
      .attr('fill', color);

    legend.append('text')
      .attr('x', 24)
      .attr('y', 9)
      .attr('dy', '.35em')
      .text(d => d);
  });
</script>

<style>
  .chart :global(rect) {
    stroke: #fff;
  }
</style>

<div bind:this={el} class="chart"></div>
 -->
