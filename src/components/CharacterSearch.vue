<template>
	<div class="search">
		<div class="search-container">
			<label for="character-search-1">
				<input
					type="text"
					class="search-input"
					id="character-search-1"
					v-model="searchQuery1"
					@input="filterCharacters(1)"
				/>
				<button
					v-if="searchQuery1"
					class="clear-button-1"
					@click="clearSearch(1)"
				>
					X
				</button>
			</label>
			<ul
				v-if="filteredCharacters1.length > 0 && searchQuery1.length > 0"
				class="suggestions"
			>
				<li
					v-for="character in filteredCharacters1"
					:key="character.name"
					@click="selectCharacter(character.name, 1)"
				>
					{{ character.name }}
				</li>
			</ul>
		</div>

		<div class="search-container">
			<label for="character-search-2">
				<input
					type="text"
					id="character-search-2"
					class="search-input"
					v-model="searchQuery2"
					@input="filterCharacters(2)"
				/>
				<button
					v-if="searchQuery2"
					class="clear-button-2"
					@click="clearSearch(2)"
				>
					X
				</button>
			</label>
			<ul
				v-if="filteredCharacters2.length > 0 && searchQuery2.length > 0"
				class="suggestions"
			>
				<li
					v-for="character in filteredCharacters2"
					:key="character.name"
					@click="selectCharacter(character.name, 2)"
				>
					{{ character.name }}
				</li>
			</ul>
		</div>
	</div>
</template>

<script>
	import axios from 'axios';

	export default {
		data() {
			return {
				allCharacters: [],
				filteredCharacters1: [],
				filteredCharacters2: [],
				searchQuery1: '',
				searchQuery2: '',
				characters: [],
			};
		},
		async mounted() {
			try {
				const response = await axios.get('https://swapi.dev/api/people/');
				this.allCharacters = response.data.results;
				// Select "Luke Skywalker" and "Darth Vader" by default
				this.selectCharacter('Luke Skywalker', 1);
				this.selectCharacter('Darth Vader', 2);
				this.$emit('data-loaded');
			} catch (error) {
				console.error('Error fetching data:', error);
			}
		},
		methods: {
			filterCharacters(inputNumber) {
				const query = inputNumber === 1 ? this.searchQuery1 : this.searchQuery2;
				const filteredCharacters = this.allCharacters.filter(character =>
					character.name.toLowerCase().includes(query.toLowerCase())
				);
				if (inputNumber === 1) {
					this.filteredCharacters1 = filteredCharacters;
				} else {
					this.filteredCharacters2 = filteredCharacters;
				}
			},
			selectCharacter(name, inputNumber) {
				const character = this.allCharacters.find(
					char => char.name.toLowerCase() === name.toLowerCase()
				);
				if (character) {
					if (inputNumber === 1) {
						this.searchQuery1 = name;
						this.filteredCharacters1 = [];
						this.characters = [character, this.characters[1]].filter(Boolean);
					} else if (inputNumber === 2) {
						this.searchQuery2 = name;
						this.filteredCharacters2 = [];
						this.characters = [this.characters[0], character].filter(Boolean);
					}
					this.$emit('character-selected', { character, inputNumber });
				}
			},
			clearSearch(inputNumber) {
				if (inputNumber === 1) {
					this.searchQuery1 = '';
					this.filteredCharacters1 = [];
				} else if (inputNumber === 2) {
					this.searchQuery2 = '';
					this.filteredCharacters2 = [];
				}
			},
		},
	};
</script>

<style>
	label {
		display: block;
		border: 2px solid yellow;
		padding: 4px;
		border-radius: 15px;
	}
	.search {
		display: flex;
		gap: 20px;
	}
	.search-input {
		flex: 1;
		border: none;
		background: black;
		padding: 10px;
		outline: none;
	}
	.clear-button-1 {
		background: #43a5cf;
	}
	.clear-button-2 {
		background: #e42f43;
	}
	.clear-button-1,
	.clear-button-2 {
		border-radius: 10px;
		opacity: 1;
	}
	.clear-button-1:hover,
	.clear-button-2:hover {
		opacity: 0.8;
	}
	.search-container {
		position: relative;
		margin-bottom: 20px;
	}

	.suggestions {
		list-style-type: none;
		padding: 0;
		margin: 0;
		border: 2px solid yellow;
		max-height: 150px;
		overflow-y: auto;
		position: absolute;
		width: 100%;
		z-index: 10;
		background: white;
		color: black;
	}

	.suggestions li {
		padding: 8px;
		cursor: pointer;
	}

	.suggestions li:hover {
		background-color: black;
	}
</style>
