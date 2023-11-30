<!-- <script>
  import { onMount, onDestroy } from 'svelte';
  import * as d3 from 'd3';
  import dataset from './dataset.json';
  import { selectedPlayer } from '../lib/playerStore';

  let el;

  onMount(() => {
    updateChart();
  });

  onDestroy(() => {
    // Cleanup code if needed
  });

  // Update chart on selectedPlayer change
  $: updateChart();

  function updateChart() {
    const player = $selectedPlayer;
    const filteredData = dataset.filter((d) => d.Player === player);

    const data = [
      { label: 'Goals', value: d3.sum(filteredData, (d) => d.Liga_Goals) },
      { label: 'Assists', value: d3.sum(filteredData, (d) => d.Liga_Asts) },
      { label: 'Appearances', value: d3.sum(filteredData, (d) => d.Liga_Aps) },
    ];

    // Remove previous chart
    d3.select(el).selectAll('*').remove();

    const width = 300;
    const height = 300;
    const radius = Math.min(width, height) / 2;

    const color = d3.scaleOrdinal().range(['#F7B5CD', '#231F20', '#8da0cb']);

    const svg = d3.select(el).append('svg').attr('width', width).attr('height', height);

    const g = svg.append('g').attr('transform', `translate(${width / 2},${height / 2})`);

    const pie = d3.pie().value((d) => d.value);

    const path = d3.arc().outerRadius(radius).innerRadius(0);

    const arc = g
      .selectAll('arc')
      .data(pie(data))
      .enter()
      .append('g')
      .attr('class', 'arc');

    arc
      .append('path')
      .attr('d', path)
      .attr('fill', (d) => color(d.data.label))
      .attr('stroke', '#fff')
      .style('stroke-width', '2px');

    arc
      .append('text')
      .attr('transform', (d) => `translate(${path.centroid(d)})`)
      .attr('dy', '0.35em')
      .style('text-anchor', 'middle')
      .text((d) => d.data.label);
  }
</script>

<style>
  /* Add your styles here if needed */
</style>

<div bind:this={el}></div> -->
