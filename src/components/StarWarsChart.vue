<template>
	<canvas
		ref="chartCanvas"
		id="starwars-chart"
		class="chart"
	></canvas>
</template>

<script>
	import axios from 'axios';
	import { Chart, registerables } from 'chart.js';

	Chart.register(...registerables);

	export default {
		name: 'StarWarsChart',
		props: {
			characters: {
				type: Array,
				default: () => [],
			},
		},
		data() {
			return {
				chart: null,
				prevCharacters: [],
			};
		},
		watch: {
			characters: {
				handler(newData) {
					if (
						newData.length === 2 &&
						!this.isDataSame(newData, this.prevCharacters)
					) {
						this.prevCharacters = JSON.parse(JSON.stringify(newData));
						this.updateChart(newData);
					}
				},
				immediate: true,
				deep: true,
			},
		},
		methods: {
			createChart(data) {
				const canvas = this.$refs.chartCanvas;
				const ctx = canvas.getContext('2d');
				const formattedData = data.map(character => ({
					name: character.name,
					height: parseInt(character.height, 10),
				}));

				this.chart = new Chart(ctx, {
					type: 'bar',
					data: {
						labels: formattedData.map(d => d.name),
						datasets: [
							{
								label: 'Height',
								data: formattedData.map(d => d.height),
								backgroundColor: ['#43a5cf', '#e42f43'],
								borderColor: ['#43a5cf', '#e42f43'],
								borderWidth: 1,
								borderRadius: 10,
								barPercentage: 0.5,
								categoryPercentage: 1.0,
							},
						],
					},
					options: {
						indexAxis: 'y',
						scales: {
							x: {
								beginAtZero: true,
								ticks: {
									font: {
										size: 16,
										family: 'Libre Franklin, sans-serif',
									},
									color: '#ffe81f',
									callback: function (value) {
										const totalInches = value * 0.393701;
										const feet = Math.floor(totalInches / 12);
										const inches = Math.round(totalInches % 12);
										return `${feet}' ${inches}"`;
									},
								},
								grid: {
									color: '#ffe81f',
								},
							},
							y: {
								ticks: {
									font: {
										size: 16,
										family: 'Libre Franklin, sans-serif',
									},
									color: '#ffe81f',
								},
								grid: {
									color: '#ffe81f',
								},
							},
						},
						plugins: {
							legend: {
								display: false,
							},
						},
					},
				});
			},
			updateChart(data) {
				this.$nextTick(() => {
					this.clearCanvas();
					this.destroyChart();
					this.createChart(data);
				});
			},
			destroyChart() {
				if (this.chart) {
					this.chart.destroy();
					this.chart = null;
				}
			},
			clearCanvas() {
				const canvas = this.$refs.chartCanvas;
				if (canvas) {
					const ctx = canvas.getContext('2d');
					if (ctx) {
						ctx.clearRect(0, 0, canvas.width, canvas.height);
					}
				}
			},
			isDataSame(newData, oldData) {
				return JSON.stringify(newData) === JSON.stringify(oldData);
			},
		},
		beforeUnmount() {
			this.destroyChart();
		},
	};
</script>

<style>
	#starwars-chart {
		width: 100%;
		height: 400px;
		border: 2px solid yellow;
		padding: 20px;
	}
</style>
