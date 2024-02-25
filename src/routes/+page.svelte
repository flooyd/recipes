<script lang="ts">
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import { fade } from 'svelte/transition';
	import { v4 as uuid } from 'uuid';

	const recipes = writable<any>([]);
	const selectedRecipe = writable<string>('');
	let pageLoading = true;
	let recipeLoading = false;

	const fetchRecipes = async () => {
		const res = await fetch('http://localhost:3000/recipes/all');
		const data = await res.json();
		return data;
	};

	const createRecipe = async () => {
		recipeLoading = true;
		const res = await fetch('http://localhost:3000/recipes/create', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				name: uuid(),
				ingredients: ['Test Ingredient 1', 'Test Ingredient 2'],
				instructions: ['Test Step 1', 'Test Step 2']
			})
		});
		const data = await res.json();
		$recipes = [...$recipes, data];
		recipeLoading = false;
	};

	onMount(async () => {
		$recipes = await fetchRecipes();
		pageLoading = false;
	});
</script>

<nav>
	<div class="title">The Recipe Site</div>
	<div class="options">
		<div>About</div>
		<div>Login</div>
	</div>
</nav>

<main>
	<div class="intro">
		Welcome to the recipe site. Here you can create and share recipes with your friends.
	</div>
	<div class="recipes">
		<div class="recipesTitle">Recipes</div>
		<div class="toolbar">
			<button disabled={recipeLoading} on:click={createRecipe}>Create Recipe</button>
		</div>
		{#each $recipes as recipe (recipe.name)}
			<button
				class="recipeName"
				on:click={() => ($selectedRecipe = recipe.name)}
				on:keydown={(event) => {
					if (event.key === 'Enter') $selectedRecipe = recipe.name;
				}}>{recipe.name}
            </button>
            {#if $selectedRecipe === recipe.name}
                <div>
                    <div>Ingredients:</div>
                    <ul>
                        {#each recipe.ingredients as ingredient}
                            <li>{ingredient}</li>
                        {/each}
                    </ul>
                    <div>Steps:</div>
                    <ol>
                        {#each recipe.instructions as step}
                            <li>{step}</li>
                        {/each}
                    </ol>
                </div>
            {/if}
		{/each}
	</div>
</main>

<style>
	:global(*) {
		box-sizing: border-box;
		font-family: 'Inconsolata', monospace;
	}

	nav {
		background: lightcoral;
		height: 40px;
		font-size: 25px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0px 20px;
		font-family: 'Merriweather', serif;
	}

	.title {
		font-weight: bold;
	}

	.options {
		display: flex;
		gap: 10px;
		font-size: 20px;
	}

	.intro {
		background: #333;
		width: fit-content;
		padding: 5px;
		color: white;
		border-radius: 5px;
		margin-bottom: 15px;
		font-size: 15px;
	}

	.recipes {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	.recipesTitle {
		font-size: 20px;
		font-weight: bold;
		border-bottom: 3px solid lightcoral;
		padding-bottom: 5px;
		width: fit-content;
	}

	main {
		padding: 20px;
	}

	button {
		padding: 5px;
		border-radius: 5px;
		background: lightcoral;
		color: black;
		border: none;
		cursor: pointer;
		text-align: left;
	}

	button:disabled {
		background: lightgray;
		cursor: not-allowed;
	}

	button:hover {
		background: darkred;
		color: white;
	}

	.recipeName {
		font-size: 20px;
		font-weight: bold;
	}
</style>
