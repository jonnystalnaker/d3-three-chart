<template>
	<div id="far-far-away">
		<img
			src="./assets/swlogo.png"
			alt="Star Wars logo"
		/>
		<h1>Star Wars Height Check</h1>
		<h2>Who's the tallest?</h2>
		<p>
			This is a simple app that lets you check the height of characters from the
			Star Wars universe. It uses D3.js, Three.js, and Chart.js to create a 3D
			bar chart and a 2D bar chart, respectively. It calls the
			https://swapi.dev/ api to get Starwars Characters
		</p>
		<p>
			Repo can be found at
			<a
				target="_blank"
				href="https://github.com/jonnystalnaker/d3-three-chart"
			>
				https://github.com/jonnystalnaker/d3-three-chart
			</a>
		</p>
		<CharacterSearch
			@character-selected="updateCharacters"
			@data-loaded="dataLoaded"
		/>
		<div v-if="!loading">
			<div class="tabs">
				<button
					v-for="tab in tabs"
					:key="tab"
					:class="{ active: activeTab === tab }"
					@click="activeTab = tab"
				>
					{{ tab }}
				</button>
			</div>
			<div class="chart-container">
				<div
					class="charts"
					:class="{ active: activeTab === 'D3.js' }"
				>
					<StarWarsD3 :characters="characters" />
				</div>
				<div
					class="charts"
					:class="{ active: activeTab === 'Three.js' }"
				>
					<StarWarsThree :characters="characters" />
				</div>
				<div
					class="charts"
					:class="{ active: activeTab === 'Chart.js' }"
				>
					<StarWarsChart :characters="characters" />
				</div>
			</div>
		</div>
		<div v-else>Loading...</div>
	</div>
</template>

<script>
	import CharacterSearch from './components/CharacterSearch.vue';
	import StarWarsD3 from './components/StarWarsD3.vue';
	import StarWarsThree from './components/StarWarsThree.vue';
	import StarWarsChart from './components/StarWarsChart.vue';

	export default {
		name: 'App',
		components: {
			CharacterSearch,
			StarWarsD3,
			StarWarsThree,
			StarWarsChart,
		},
		data() {
			return {
				characters: [],
				tabs: ['D3.js', 'Three.js', 'Chart.js'],
				activeTab: 'D3.js',
				loading: true,
			};
		},
		methods: {
			async updateCharacters({ character, inputNumber }) {
				if (inputNumber === 1) {
					this.characters[0] = character;
				} else if (inputNumber === 2) {
					this.characters[1] = character;
				}
				this.characters = [...this.characters]; // Ensure reactivity
			},
			dataLoaded() {
				this.loading = false;
			},
		},
	};
</script>

<style>
	#far-far-away {
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: center;
		z-index: 1000;
		width: 820px;
	}
	h1 {
		margin-bottom: 0;
	}
	h2,
	p {
		color: yellow;
	}
	#far-far-away img {
		width: 400px;
		margin-bottom: 20px;
	}
	.tabs {
		display: flex;
		justify-content: center;
		margin-bottom: 20px;
	}
	.tabs button {
		padding: 10px 20px;
		cursor: pointer;
		background-color: #222;
		color: #fff;
		border: none;
		margin-right: 10px;
	}
	.tabs button.active {
		background-color: #444;
		color: #ffe81f;
	}
	.chart-container {
		width: 810px;
		height: 410px;
		position: relative;
	}
	.charts {
		width: 100%;
		height: 100%;
		position: absolute;
		top: 0;
		left: 0;
		opacity: 0;
		transition: opacity 0.3s ease;
	}
	.charts.active {
		opacity: 1;
		z-index: 1;
	}
</style>
