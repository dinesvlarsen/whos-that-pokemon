<template>
	<div class="guess-pokemon">
		<div class="guess-pokemon__image">
			<img
				:src="correctPokemon.image.url"
				:alt="correctPokemon.image.altText"
			/>
		</div>

		<h1 class="pokemon-font">{{ pokemonOutput }}</h1>
	</div>
</template>

<script>
export default {
	data() {
		return {
			correctPokemon: {
				image: {
					url: '',
					altText: 'Picture of a pokemon',
				},
				index: null,
				name: '',
			},
			pokemonOutput: "who's that Pokemon?",
			gamePokemons: [],
			correctButtonPressed: false,
			randomIdsUsed: [], //Array used to keep track of what pokemons has been fetched
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
		async fetchPokemon() {
			const url = 'https://pokeapi.co/api/v2/pokemon?limit=151';
			const response = await fetch(url);
			const { results } = await response.json();
			this.allPokemons = results;
		},

		//Method used to start the app.
		runApp() {
			this.insertRandomPokemons();
			this.getCorrectPokemon();
		},

		//Gets the pokemon used as the correct answer, and sets the image and name of that pokemon.
		getCorrectPokemon() {
			const correctPokemonIndex = this.randomButtonNumber();
			const correctPokemon = this.gamePokemons[correctPokemonIndex];

			this.getCorrectPokemonImage(correctPokemon.url);
			this.setCorrectPokemonData(correctPokemon.name, correctPokemonIndex);
		},

		async getCorrectPokemonImage(correctPokemonUrl) {
			const url = correctPokemonUrl;
			const response = await fetch(url);
			const pokemonData = await response.json();

			const pokemonImage = pokemonData.sprites.front_default;
			this.setPokemonImage(pokemonImage);
		},

		setCorrectPokemonData(correctPokemonName, correctPokemonIndex) {
			this.correctPokemon.name = correctPokemonName;
			this.correctPokemon.index = correctPokemonIndex;
		},

		//Gets a random pokemon from the pokeApi and pushes it to the gamePokemons array.
		getRandomPokemonNames() {
			const randomIndex = this.randomIndex();

			if (this.randomIdsUsed.includes(randomIndex)) {
				this.getRandomPokemonName();
			} else {
				this.gamePokemons.push(this.allPokemons[randomIndex]);

				//Pushes the id used to the randomIdsUsed array, so we can keep track of all the ids used.
				this.randomIdsUsed.push(randomIndex);
			}
		},

		setPokemonImage(pokemonImageUrl) {
			this.correctPokemon.image.url = pokemonImageUrl;
		},

		insertRandomPokemons() {
			for (let i = 0; 4 > i; i++) {
				this.getRandomPokemonNames();
			}
		},

		//Generates a random number used for pokemonId, the number is used in the pokeapi http link to get a random pokemon.
		randomIndex() {
			let randomId = Math.floor(Math.random() * 150);
			return randomId;
		},

		//Capitalizes the first character of a string. (Used to capitalize the names from the pokeapi, since they were are all lowercase by default)
		capitalizeString(string) {
			return string.charAt(0).toUpperCase() + string.slice(1);
		},

		//Generates a random number between 0 and 3 which is used to determine which button to insert the correct pokemon name.
		randomButtonNumber() {
			return Math.floor(Math.random() * 3);
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
