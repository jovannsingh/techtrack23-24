<script>
	import { onMount, afterUpdate } from 'svelte';
	import * as d3 from 'd3';
	import dataset from './dataset.json';
	import { selectedPlayer } from '../lib/playerStore';

	let elBarChart;
	let elPieChart;
	let elLineChart;

	// Initial chart setup
	onMount(() => {
		updateChart($selectedPlayer); // Het roept de functie updateChart($selectedPlayer) aan om het diagram in te stellen op basis van de momenteel geselecteerde speler
	});

	afterUpdate(() => {
		updateChart($selectedPlayer); // Het roept de functie updateChart($selectedPlayer) aan om het diagram in te stellen op basis van de momenteel geselecteerde speler
	});

	// Update het diagram bij wijziging van geselecteerde speler
	function updateChart(player) {
		d3.select(elBarChart).selectAll('*').remove();
		d3.select(elPieChart).selectAll('*').remove();
		d3.select(elLineChart).selectAll('*').remove();

		const margin = { top: 60, right: 20, bottom: 30, left: 40 };
		const width = 800 - margin.left - margin.right;
		const height = 500 - margin.top - margin.bottom;

		// Bar Chart
		const svgBarChart = d3
			.select(elBarChart)
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr('transform', `translate(${margin.left},${margin.top})`);

		const x = d3.scaleBand().range([0, width]).padding(0.1);
		const y = d3.scaleLinear().range([height, 0]);

		const filteredData = dataset.filter((d) => d.Player === player); //// Filter de dataset op de geselecteerde speler
		const seasons = Array.from(new Set(filteredData.map((d) => d.Season)));

		x.domain(seasons); // Stel het domein in voor de x-as (seizoenen)
		y.domain([
			// Stel het domein in voor de y-as (doelpunten)
			0,
			d3.max(
				filteredData,
				(d) => d.Liga_Goals + d.CL_Goals,
				(d) => d.Liga_Aps + d.CL_Aps
			)
		]);

		// Bar Chart
		svgBarChart
			.selectAll('rect.goals')
			.data(filteredData)
			.enter()
			.append('rect')
			.attr('class', 'goals')
			.style('width', x.bandwidth() / 2)
			.style('height', (d) => height - y(d.Liga_Goals + d.CL_Goals))
			.style('x', (d) => x(d.Season))
			.style('y', (d) => y(d.Liga_Goals + d.CL_Goals))
			.style('fill', '#FEBE10') // Custom kleur voor Goals

			// Voeg hover effect
			.on('mouseover', function (event, d) {
				d3.select(this)
					.transition()
					.duration(200)
					.style('fill', '#4CAF50')
					.attr('height', (d) => height - y(d.Liga_Goals + d.CL_Goals) + 10);
			})
			.on('mouseout', function (event, d) {
				d3.select(this)
					.transition()
					.duration(200)
					.style('fill', '#FEBE10')
					.attr('height', (d) => height - y(d.Liga_Goals + d.CL_Goals));
			});

		svgBarChart
			.selectAll('rect.appearances')
			.data(filteredData)
			.enter()
			.append('rect')
			.attr('class', 'appearances')
			.style('width', x.bandwidth() / 2)
			.style('height', (d) => height - y(d.Liga_Aps + d.CL_Aps))
			.style('x', (d) => x(d.Season) + x.bandwidth() / 2)
			.style('y', (d) => y(d.Liga_Aps + d.CL_Aps))
			.style('fill', '#38423B'); // Custom kleur voor Appearances

		// Axes voor Bar Chart
		svgBarChart.append('g').attr('transform', `translate(0, ${height})`).call(d3.axisBottom(x));

		svgBarChart.append('g').call(d3.axisLeft(y));

		// Titel voor Bar Chart
		svgBarChart
			.append('text')
			.attr('x', width / 4)
			.attr('y', -margin.top / 2)
			.attr('text-anchor', 'middle')
			.style('font-family', 'Roboto, sans-serif')
			.style('font-weight', 'bold')
			.style('fill', '#FEBE10')
			.text('Goals');

		svgBarChart
			.append('text')
			.attr('x', (width / 4) * 3)
			.attr('y', -margin.top / 2)
			.attr('text-anchor', 'middle')
			.style('font-family', 'Roboto, sans-serif')
			.style('font-weight', 'bold')
			.style('fill', '#38423B')
			.text('Appearances');

		// Pie Chart
		const pieData = [
			{ label: 'Liga Goals', value: filteredData.reduce((acc, d) => acc + d.Liga_Goals, 0) },
			{ label: 'Liga Assists', value: filteredData.reduce((acc, d) => acc + d.Liga_Asts, 0) },
			{ label: 'CL Goals', value: filteredData.reduce((acc, d) => acc + d.CL_Goals, 0) },
			{ label: 'CL Assists', value: filteredData.reduce((acc, d) => acc + d.CL_Asts, 0) }
		];

		const pie = d3.pie().value((d) => d.value);
		const arc = d3
			.arc()
			.innerRadius(0)
			.outerRadius(Math.min(width, height) / 2);

		const svgPieChart = d3
			.select(elPieChart)
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr(
				'transform',
				`translate(${(width + margin.left + margin.right) / 2},${
					(height + margin.top + margin.bottom) / 2
				})`
			);

		svgPieChart
			.selectAll('path.pie')
			.data(pie(pieData))
			.enter()
			.append('path')
			.attr('class', 'pie')
			.attr('d', arc)
			.each(function (_, i) {
				const originalColor = d3.schemeCategory10[i % 10];
				d3.select(this).attr('originalColor', originalColor);
			})
			.style('fill', (_, i) => d3.schemeCategory10[i % 10])
			.on('mouseover', function (event, d) {
				d3.select(this)
					.transition()
					.duration(200)
					.style('fill', 'lightblue')
					.attr('transform', 'scale(1.1)');
			})
			.on('mouseout', function (event) {
				const originalColor = d3.select(this).attr('originalColor');
				d3.select(this)
					.transition()
					.duration(200)
					.style('fill', originalColor)
					.attr('transform', 'scale(1)');
			});

		svgPieChart
			.selectAll('text.label')
			.data(pie(pieData))
			.enter()
			.append('text')
			.attr('class', 'label')
			.attr('transform', (d) => `translate(${arc.centroid(d)})`)
			.attr('dy', '0.35em')
			.style('text-anchor', 'middle')
			.style('font-family', 'Roboto, sans-serif')
			.style('font-size', '12px')
			.style('fill', '#fff')
			.text((d) => d.data.value);

		// Legenda voor Pie Chart
		const legendContainer = d3
			.select(elPieChart)
			.append('div')
			.attr('class', 'legend-container')
			.style('position', 'absolute')
			.style('top', '750px')
			.style('right', '530px');

		const legendItems = legendContainer
			.selectAll('.legend-item')
			.data(pieData)
			.enter()
			.append('div')
			.attr('class', 'legend-item')
			.style('display', 'flex')
			.style('align-items', 'center')
			.style('margin-bottom', '8px');

		legendItems
			.append('div')
			.attr('class', 'legend-color')
			.style('width', '12px')
			.style('height', '12px')
			.style('background-color', (_, i) => d3.schemeCategory10[i % 10]);

		legendItems
			.append('span')
			.text((d) => `${d.label} (${d.value})`)
			.style('margin-left', '4px')
			.style('font-family', 'Roboto, sans-serif')
			.style('font-size', '12px')
			.style('color', '#333');

		// Line Chart
		const svgLineChart = d3
			.select(elLineChart)
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr('transform', `translate(${margin.left},${margin.top})`);

		const line = d3
			.line()
			.x((d) => x(d.Season))
			.y((d) => y(d.Market_Value));

		x.domain(seasons);
		y.domain([0, d3.max(filteredData, (d) => d.Market_Value)]);

		// Voeg een groepselement toe voor de cirkels
		const circleGroup = svgLineChart.append('g');

		// Voeg de cirkels toe aan de groep
		circleGroup
			.selectAll('circle')
			.data(filteredData)
			.enter()
			.append('circle')
			.attr('cx', (d) => x(d.Season))
			.attr('cy', (d) => y(d.Market_Value))
			.attr('r', 5)
			.attr('fill', '#4CAF50'); // Kleur van de cirkels

		// Voeg de datalijn toe
		svgLineChart
			.append('path')
			.datum(filteredData)
			.attr('class', 'line')
			.attr('d', line)
			.attr('fill', 'none');

		svgLineChart
			.append('path')
			.datum(filteredData)
			.attr('d', line)
			.attr('fill', 'none')
			.attr('stroke', '#4CAF50')
			.attr('stroke-width', 2);

		// Assen for Line Chart
		svgLineChart.append('g').attr('transform', `translate(0, ${height})`).call(d3.axisBottom(x));

		svgLineChart.append('g').call(d3.axisLeft(y));

		// Titel voor Line Chart
		svgLineChart
			.append('text')
			.attr('x', width / 2)
			.attr('y', -margin.top / 2)
			.attr('text-anchor', 'middle')
			.style('fill', '#4CAF50');
	}
</script>

<!-- HTML Gedeelte -->

<div>
	<h1>Who's the GOAT? Check the stats</h1>
</div>

<div class="image-wrapper">
	<!-- Messi's afbeelding -->
	<img
		class={$selectedPlayer === 'Messi' ? 'glow messi' : 'messi'}
		src="https://www.transparentpng.com/download/messi/jCQ4CA-messi-wins-barcelona-the-milestone-match-steemit.png"
		alt="Messi"
	/>

	<!-- Ronaldo's afbeelding -->
	<img
		class={$selectedPlayer === 'Ronaldo' ? 'glow ronaldo' : 'ronaldo'}
		src="https://freepngimg.com/thumb/cristiano_ronaldo/21842-8-cristiano-ronaldo.png"
		alt="Ronaldo"
	/>
</div>

<div class="allescenter">
	<!-- Dropdown -->
	<select bind:value={$selectedPlayer}>
		<option value="Messi">Messi</option>
		<option value="Ronaldo">Ronaldo</option>
	</select>

	<!-- Bar Chart Container -->
	<div bind:this={elBarChart} />

	<!-- Pie Chart Title -->
	<h2>Liga and Champions League in Goals and Appearances</h2>

	<!-- Pie Chart Container -->
	<div bind:this={elPieChart} />

	<!-- Line Chart Title -->
	<h2>Market Value over Time</h2>

	<!-- Line Chart Container -->
	<div bind:this={elLineChart} />
</div>

<style>
	h1 {
		font-family: 'Roboto', sans-serif;
		text-align: center;
		margin-top: 50px;
		font-size: 24px;
		font-weight: bold;
		color: #333;
	}

	h2 {
		font-family: 'Roboto', sans-serif;
		text-align: center;
		font-size: 20px;
		font-weight: bold;
		color: #333;
	}

	.image-wrapper {
		display: flex;
		justify-content: space-between;
		margin: 20px;
		position: relative;
		top: 255px;
	}

	img {
		width: 250px;
		height: auto;
		display: block;
		transition: filter 0.3s ease-in-out;
	}

	img.messi.glow {
		filter: drop-shadow(0 0 10px #234cff);
	}

	img.ronaldo.glow {
		filter: drop-shadow(0 0 10px #ff9000);
	}

	.allescenter {
		text-align: center;
		margin-top: -150px;
		position: relative;
		top: -250px;
	}
</style>
