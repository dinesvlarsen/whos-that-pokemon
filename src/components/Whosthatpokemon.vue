<template>
	<div class="guess-pokemon">
		<div class="guess-pokemon__image">
			<img
				:class="{ 'show-image': correctButtonStatus }"
				:src="correctPokemon.image.url"
				:alt="correctPokemon.image.altText"
			/>
		</div>

		<h1 class="guess-pokemon__output">{{ pokemonOutput }}</h1>

		<!-- Was struggling with the button logic here, ended up going for an approach found here https://stackoverflow.com/questions/43956662/event-handling-dynamic-created-buttons-in-vue-js-v-for-loop where u add a boolean property to the objects used for the buttons. -->
		<div class="guess-pokemon__buttons">
			<button
				@click="buttonOnClick(buttonIndex)"
				class="guess-pokemon__button"
				:class="{
					'guess-pokemon__button--incorrect': gamePokemons[buttonIndex].clicked,
					//prettier-ignore
					'guess-pokemon__button--correct':gamePokemons[buttonIndex].correctClicked,
				}"
				v-for="(pokemon, buttonIndex) in this.gamePokemons"
				:key="pokemon.buttonIndex"
			>
				<div>{{ pokemon.name }}</div>
			</button>
		</div>

		<button
			class="guess-pokemon__decorative-sprite guess-pokemon__decorative-sprite--hidden"
			@click="spriteOnClick()"
		>
			<img :src="decorativeSprite.url" :alt="decorativeSprite.altText" />
		</button>
	</div>
</template>

<script>
export default {
	data() {
		return {
			correctButtonStatus: false,
			correctPokemon: {
				name: '',
				correctAnswerIndex: null,
				image: {
					url: '',
					altText: 'Picture of a pokemon',
				},
			},
			pokemonOutput: "Who's that Pokemon?",
			gamePokemons: [],
			randomIndexesUsed: [], //Keeps track of what pokemons has been pushed into gamePokemons.
			allPokemons: [],
			decorativeSprite: {
				url: '',
				altText: '',
			},
		};
	},

	async created() {
		if (this.allPokemons.length > 0) {
			this.fetchPokemon();
		}

		this.fetchPokemon().finally(() => {
			this.runApp();
			this.setDecorativeSprite();
		});
	},

	methods: {
		//Click function
		buttonOnClick(buttonIndex) {
			//Guard clause to stop clicking once correct button has been found.
			if (this.correctButtonStatus) return;

			if (buttonIndex === this.correctPokemon.correctAnswerIndex) {
				this.correctButtonClicked(buttonIndex);
			} else {
				this.gamePokemons[buttonIndex].clicked = true;
			}
		},

		spriteOnClick() {
			this.setDecorativeSprite();
		},

		//Used when the correct button is clicked.
		correctButtonClicked(buttonIndex) {
			this.correctButtonStatus = true;
			this.gamePokemons[buttonIndex].correctClicked = true;
			this.pokemonOutput = `It's ${this.correctPokemon.name}!`;

			//Reruns the app after 2 seconds to load new pokemons.
			setTimeout(() => {
				this.runApp();
			}, 2000);
		},

		//Method used to start the app.
		runApp() {
			//Checks if the game has gone through its first game, and if it has resets all the data cached.
			if (this.gamePokemons.length > 0) this.resetApp();

			this.getFourRandomPokemons();

			// This sets all the data we need for the correctPokemon.
			this.setCorrectPokemonData();
		},

		//Resets the data used in the app.
		resetApp() {
			//Deletes the clicked and correctClicked properties from the pokemons retrieved, -
			// - since these properties also gets applied to the objects in allPokemons.
			this.gamePokemons.forEach((pokemon) => {
				delete pokemon.clicked;
				delete pokemon.correctClicked;
			});
			this.gamePokemons = [];
			this.correctButtonStatus = false;
			this.randomIndexesUsed = [];
			this.pokemonOutput = "Who's that Pokemon?";
		},

		//Fetches all the pokemons from the pokeApi and caches them.
		async fetchPokemon() {
			try {
				const url = 'https://pokeapi.co/api/v2/pokemon?limit=151';
				const response = await fetch(url);
				const { results } = await response.json();

				this.allPokemons = results;
				this.mutateAllPokemons();
			} catch (error) {
				//Solution for a dynamic img src with vite taken from https://stackoverflow.com/questions/66419471/vue-3-vite-dynamic-img-src
				this.correctPokemon.image.url = new URL(
					'./../assets/images/error.png',
					import.meta.url
				).href;
				this.pokemonOutput = `Something went wrong! ${error}`;

				console.error(
					'⚠️⚠️⚠️⚠️⚠️⚠️ERROR ERROR ERROR DUDE LOOK HERE ERROR!⚠️⚠️⚠️⚠️⚠️' +
						error
				);
				console.error(error.message);
			}
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

			//Sets seperate onclick boolean value on the object in gamePokemons that has the correct pokemon
			this.gamePokemons.forEach((pokemon) => {
				if (pokemon === this.gamePokemons[correctPokemonIndex]) {
					delete pokemon.clicked;
					pokemon.correctClicked = false;
				}
			});
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
			const randomIndexesUsed = this.randomIndexesUsed.includes(randomIndex);

			if (randomIndexesUsed) {
				this.setGamePokemons();
			} else {
				const pokemon = this.allPokemons[randomIndex];
				pokemon.clicked = false;
				pokemon.name = this.capitalizeString(pokemon.name);
				// this.allPokemon[randomIndex].pokemonIndex = randomIndex + 1;
				this.gamePokemons.push(pokemon);

				//Pushes the id used to the randomIndexesUsed array, so we can keep track of all the ids used.
				this.randomIndexesUsed.push(randomIndex);
			}
		},

		//Creates a random pokemon image link to use as a decorativeSprite.url.
		setDecorativeSprite() {
			const randomIndex = this.randomIndex();

			//Solution for a dynamic img src with vite taken from https://stackoverflow.com/questions/66419471/vue-3-vite-dynamic-img-src
			const decorativeSprite = new URL(
				`./../assets/images/poke-sprites-kanto/${randomIndex + 1}.png`,
				import.meta.url
			).href;
			const decorativeSpriteAltText = `A decorative picture of ${this.allPokemons[randomIndex].name}`;

			this.decorativeSprite.url = decorativeSprite;
			this.decorativeSprite.altText = decorativeSpriteAltText;
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
.guess-pokemon {
	max-width: 1440px;
	margin: var(--l) var(--m) 0 var(--m);
}

.guess-pokemon__image {
	background-color: #62ddd5;
	border-radius: 50%;
	border: 8px solid var(--primary);
	margin: 0 auto;
	max-width: 199px;
}

.guess-pokemon__image img {
	filter: brightness(0%);
}

@keyframes showImage {
	from {
		filter: brightness(10%);
	}
	to {
		filter: brightness(100%);
	}
}
.show-image {
	filter: brightness(100%);
	animation-name: showImage;
	animation: showImage 2s forwards;
}

.guess-pokemon__output {
	color: #ffcb05;
	-webkit-text-stroke-width: 1px;
	-webkit-text-stroke-color: var(--secondary);
	font-size: var(--output-font-size);
	letter-spacing: 0.163em;
	font-family: var(--output-font-family);
	text-align: center;
	margin-top: var(--m);
}

.guess-pokemon__button {
	background-color: #ffcb05;
	display: block;
	color: rgb(54, 54, 54);
	width: 100%;
	max-width: 342px;
	padding: 14px;
	margin: var(--s) auto;
	border-radius: 4px;
	border: solid 2px rgba(0, 10, 66, 0.61);
	font-size: var(--button-font-size);
}

.guess-pokemon__button--correct {
	background-color: var(--correct);
}

.guess-pokemon__button--incorrect {
	background-color: var(--incorrect);
	text-decoration: line-through;
}

.guess-pokemon__decorative-sprite--hidden {
	display: none;
}

.guess-pokemon__decorative-sprite {
	position: relative;
	bottom: 0;
	left: 0;
	width: 200px;
}

/* MEDIA QUERIES MOBILE FIRST */

@media screen and (min-width: 744px) {
	.guess-pokemon {
		margin: 116px 24px 0 24px;
	}

	.guess-pokemon__image {
		max-width: 303px;
		margin-top: var(--l);
	}

	.guess-pokemon__output {
		-webkit-text-stroke-width: 2px;
		margin-top: var(--xl);
		margin-bottom: var(--l);
	}

	.guess-pokemon__button {
		margin-top: 0;
		border: solid 3px var(--secondary);
		max-width: 398px;
	}

	.guess-pokemon__decorative-sprite--hidden {
		display: block;
	}
}

@media screen and (min-width: 1440px) {
	.guess-pokemon {
		margin-left: auto;
		margin-right: auto;
	}

	.guess-pokemon__buttons {
		display: grid;
		grid-template-columns: minmax(398px, auto) minmax(398px, auto);
		grid-gap: var(--s) var(--xxl);
		margin: 0 auto;
		justify-content: center;
	}

	.guess-pokemon__decorative-sprite {
		position: absolute;
	}
}
</style>
