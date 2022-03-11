<template>
	<div class="guess-pokemon">
		<div class="guess-pokemon__image">
			<img
				:src="correctPokemon.image.url"
				:alt="correctPokemon.image.altText"
			/>
		</div>

		<h1 class="pokemon-font">{{ pokemonOutput }}</h1>

		<button
			@click.once="onClickCheck(index)"
			class="guess-pokemon__buttons"
			v-for="(pokemon, index) in this.gamePokemons"
			:key="pokemon.index"
		>
			<div>{{ pokemon.name }}</div>
		</button>
	</div>
</template>

<script>
export default {
	data() {
		return {
			correctPokemon: {
				name: '',
				correctAnswerIndex: null,
				image: {
					url: '',
					altText: 'Picture of a pokemon',
				},
			},
			pokemonOutput: "who's that Pokemon?",
			gamePokemons: [],
			correctButtonPressed: false,
			randomIndexesUsed: [], //Array used to keep track of what pokemons has been fetched
			allPokemons: [],
		};
	},

	async created() {
		if (this.allPokemons.length > 0) {
			this.fetchPokemon();
		}

		this.fetchPokemon().finally(() => {
			this.runApp();
		});
	},

	methods: {
		//Method used to start the app.
		runApp() {
			this.getFourRandomPokemons();

			// This sets all the data we need for the correctPokemon.
			this.setCorrectPokemonData();
		},

		//Fetches all the pokemons from the pokeApi and caches them.
		async fetchPokemon() {
			const url = 'https://pokeapi.co/api/v2/pokemon?limit=151';
			const response = await fetch(url);
			const { results } = await response.json();
			this.allPokemons = results;
			this.mutateAllPokemons();
		},

		//Adds an index to each pokemon in allPokemons, this index is used to get images from assets/images/poke-sprites-kanto folder.
		mutateAllPokemons() {
			this.allPokemons.forEach((pokemon, index) => {
				//Adding +1 to the index because the image urls start from 1 and not 0.
				pokemon.index = index + 1;
			});
		},

		//Sets all the data needed for correctPokemon.
		setCorrectPokemonData() {
			//Variables used to set data.
			const correctPokemonIndex = this.randomButtonIndex();
			// prettier-ignore
			const correctPokemonImageIndex = this.gamePokemons[correctPokemonIndex].index;
			const correctPokemonName = this.gamePokemons[correctPokemonIndex].name;

			//Solution for a dynamic img src with vite taken from https://stackoverflow.com/questions/66419471/vue-3-vite-dynamic-img-src
			const correctPokemonImage = new URL(
				`./../assets/images/poke-sprites-kanto/${correctPokemonImageIndex}.png`,
				import.meta.url
			).href;

			//Sets data on correctPokemon
			this.correctPokemon.correctAnswerIndex = correctPokemonIndex;
			this.correctPokemon.name = correctPokemonName;
			this.correctPokemon.image.url = correctPokemonImage;
		},

		//Calls setGamePokemons() four times by using a for loop.
		getFourRandomPokemons() {
			for (let i = 0; 4 > i; i++) {
				this.setGamePokemons();
			}
		},

		//Gets a random pokemon from the allPokemons and pushes it to the gamePokemons array.
		setGamePokemons() {
			const randomIndex = this.randomIndex();

			if (this.randomIndexesUsed.includes(randomIndex)) {
				this.setGamePokemons();
			} else {
				const pokemon = this.allPokemons[randomIndex];
				pokemon.name = this.capitalizeString(pokemon.name);
				// this.allPokemon[randomIndex].pokemonIndex = randomIndex + 1;
				this.gamePokemons.push(pokemon);

				//Pushes the id used to the randomIndexesUsed array, so we can keep track of all the ids used.
				this.randomIndexesUsed.push(randomIndex);
			}
		},

		//Generates a random number used for pokemonId, the number is used in the pokeapi http link to get a random pokemon.
		randomIndex() {
			let randomId = Math.floor(Math.random() * 150);
			return randomId;
		},

		//Generates a random number between 0 and 3 which is used to determine which button to insert the correct pokemon name.
		randomButtonIndex() {
			return Math.floor(Math.random() * 3);
		},

		//Capitalizes the first character of a string. (Used to capitalize the names from the pokeapi, since they were are all lowercase by default)
		capitalizeString(string) {
			return string.charAt(0).toUpperCase() + string.slice(1);
		},
	},
};
</script>

<style>
.pokemon-font {
	color: #ffcb05;
	-webkit-text-stroke-width: 0.06em;
	-webkit-text-stroke-color: #385aaa;
	letter-spacing: 4px;
	font-size: 20px;
	text-align: center;
	font-family: 'pokemon-font';
	margin-bottom: 1rem;
}

.guess-pokemon {
	margin: 1rem;
}

.guess-pokemon__image {
	background-color: #62ddd5;
	border-radius: 50%;
	border: 4px solid #ffcb05;
	width: 200px;
	margin: 30px auto;
	height: auto;
}

.guess-pokemon__image img {
	filter: brightness(0%);
}

@keyframes example {
	from {
		filter: brightness(10%);
	}
	to {
		filter: brightness(100%);
	}
}
.show {
	filter: brightness(100%);
	animation-name: example;
	animation-duration: 2s;
}

.guess-pokemon__buttons {
	background-color: #ffcb05;
	display: block;
	color: rgb(54, 54, 54);
	width: 100%;
	max-width: 450px;
	padding: 0.8em;
	margin: 0.6rem auto;
	border-radius: 4px;
	border: solid 2px rgba(0, 10, 66, 0.61);
	font-family: 'Press Start 2P', cursive;
	font-size: 14px;
}

.guess-pokemon__button--correct {
	background-color: rgb(0, 180, 0);
}

.guess-pokemon__buttons--incorrect {
	background-color: rgb(207, 41, 41);
	text-decoration: line-through;
}
</style>
