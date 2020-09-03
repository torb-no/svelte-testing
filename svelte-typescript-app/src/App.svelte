<script lang="ts">
import Game from "./Game.svelte";
import StatBar from "./StatBar.svelte";

export let name: string;

let gameActive = false;
</script>

<main>
	<h1>Hello {name}!</h1>
	<p>
		Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.
	</p>
	
	{#if gameActive}
		<Game />
		<button on:click="{() => gameActive = false}">
			End Game
		</button>
	{:else}
		<button on:click="{() => gameActive = true}">
			Start Game
		</button>
	{/if}


	
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>