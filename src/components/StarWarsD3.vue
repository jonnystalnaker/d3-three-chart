<template>
	<div
		id="d3-chart"
		class="chart"
	></div>
</template>

<script>
	import * as d3 from 'd3';

	export default {
		name: 'StarWarsD3',
		props: {
			characters: {
				type: Array,
				default: () => [],
			},
		},
		watch: {
			characters: {
				handler(newData) {
					if (newData.length === 2) {
						this.$nextTick(() => {
							this.createChart(newData);
						});
					}
				},
				immediate: true,
				deep: true,
			},
		},
		methods: {
			formatData(character) {
				if (!character || !character.name || !character.height) {
					return null;
				}
				return {
					name: character.name,
					height: parseInt(character.height, 10),
				};
			},
			createChart(data) {
				d3.select('#d3-chart').select('svg').remove();

				const formattedData = data.map(this.formatData).filter(Boolean);

				if (formattedData.length < 2) {
					return;
				}

				const margin = { top: 20, right: 30, bottom: 40, left: 180 };
				const width = 800 - margin.left - margin.right;
				const height = 400 - margin.top - margin.bottom;
				const barLeftMargin = 20;

				const svg = d3
					.select('#d3-chart')
					.append('svg')
					.attr('width', width + margin.left + margin.right)
					.attr('height', height + margin.top + margin.bottom)
					.append('g')
					.attr('transform', `translate(${margin.left},${margin.top})`);

				const y = d3
					.scaleBand()
					.range([0, height])
					.padding(0.1)
					.domain(formattedData.map(d => d.name));

				const x = d3
					.scaleLinear()
					.range([0, width])
					.domain([0, d3.max(formattedData, d => d.height)]);

				const color = d3.scaleOrdinal().range(['#43a5cf', '#e42f43']);

				const bars = svg
					.append('g')
					.selectAll('.bar')
					.data(formattedData)
					.enter()
					.append('rect')
					.attr('class', 'bar')
					.attr('y', d => y(d.name))
					.attr('height', y.bandwidth())
					.attr('x', barLeftMargin)
					.attr('width', 0)
					.attr('fill', (d, i) => color(i))
					.attr('rx', 10)
					.attr('ry', 10);

				bars
					.transition()
					.duration(1000)
					.attr('width', d => x(d.height));

				svg
					.append('g')
					.attr('class', 'x axis')
					.attr('transform', `translate(${barLeftMargin},${height})`)
					.call(d3.axisBottom(x).ticks(5).tickFormat(this.formatHeight))
					.selectAll('text')
					.style('font-size', '16px')
					.style('font-family', 'Libre Franklin, sans-serif')
					.style('fill', '#ffe81f');

				svg
					.append('g')
					.attr('class', 'y axis')
					.call(d3.axisLeft(y))
					.selectAll('text')
					.style('font-size', '16px')
					.style('font-family', 'Libre Franklin, sans-serif')
					.style('fill', '#ffe81f')
					.call(this.wrap, 100);

				svg
					.selectAll('.domain')
					.attr('stroke', '#ffe81f')
					.attr('stroke-width', 1.5);

				svg
					.selectAll('.tick line')
					.attr('stroke', '#ffe81f')
					.attr('stroke-width', 1.5);
			},
			formatHeight(cm) {
				const totalInches = cm * 0.393701;
				const feet = Math.floor(totalInches / 12);
				const inches = Math.round(totalInches % 12);
				return `${feet}' ${inches}"`;
			},
			wrap(text, width) {
				text.each(function () {
					const text = d3.select(this);
					const words = text.text().split(/\s+/).reverse();
					let word;
					let line = [];
					let lineNumber = 0;
					const lineHeight = 1.1;
					const y = text.attr('y');
					const dy = parseFloat(text.attr('dy'));
					let tspan = text
						.text(null)
						.append('tspan')
						.attr('x', -5)
						.attr('y', y)
						.attr('dy', dy + 'em');

					while ((word = words.pop())) {
						line.push(word);
						tspan.text(line.join(' '));
						if (tspan.node().getComputedTextLength() > width) {
							line.pop();
							tspan.text(line.join(' '));
							line = [word];
							tspan = text
								.append('tspan')
								.attr('x', -5)
								.attr('y', y)
								.attr('dy', ++lineNumber * lineHeight + dy + 'em')
								.text(word);
						}
					}
				});
			},
		},
	};
</script>

<style>
	#d3-chart {
		width: 100%;
		height: 400px;
		justify-content: center;
		font-family: 'Libre Franklin', sans-serif;
		border: 2px solid yellow;
		padding: 20px;
	}
</style>
