<script>
	import Square from './Square.svelte';
	import { onMount } from 'svelte/internal';
	export let gridSize = 21;
	export let TICK_DELAY_MAX = 300;
	
	let speed = 0;
	let lost = false;
	let direction = [-1, 0];


	function toTuple(num) {
		return [num % gridSize, Math.floor(num / gridSize)];
	}

	function fromTuple(tuple){
		let x = tuple[0];
		let y = tuple[1];
		return x + y * gridSize;
	}
	let squares = [];
	let food = new Set();
	food.add(fromTuple([3,3]));
	let snake = [fromTuple([10,10])];
	for(let i = 0; i < gridSize**2; i++){
		squares = [...squares, 'empty'];
	}

	squares[snake[0]] = 'snake';
	for(let f of food){
		console.log(f);
		squares[f] = 'food';
	}

	function restart() {
		squares = [];
		for(let i = 0; i < gridSize**2; i++){
			squares = [...squares, 'empty'];
		}
		snake = [fromTuple([10,10])];
		food.clear()
		food.add(fromTuple([3,3]));
		squares[snake[0]] = 'snake';
		for(let f of food){
			squares[f] = 'food';
		}
		lost = false;
	}

	function move(head){
		head = toTuple(head);
		let newHead = [head[0] + direction[0], head[1] + direction[1]]
		hasLossed(newHead);
		return fromTuple(newHead);
	}

	function hasLossed(head){
		head.forEach((coord) => {
			if(coord < 0 || coord >= gridSize){
				lost = true;
			}
		});
		if(snake.includes(fromTuple(head))){
			lost = true;
		}
	}

	function playLoop() {
		setTimeout(() => {
			let tail = snake[snake.length - 1];
			let head = move(snake[0]);
			if(food.has(head)){
				food.delete(head);
				let newFood = -1;
				do {
					newFood = Math.floor(Math.random() * gridSize**2);
				} while(snake.includes(newFood))
				console.log('newFood', newFood);
				food.add(newFood);
				squares[newFood] = 'food';
				snake = [head, ...snake];
			}
			else {
				snake = [head, ...snake.slice(0, -1)];
				squares[tail] = 'empty';
			}
			
			squares[head] = 'snake';
			
			playLoop();
		}, TICK_DELAY_MAX - speed * 20);
		
		
	}

	
	onMount(() => {
		playLoop();
	})

</script>

<svelte:window
	on:keydown={(e) => {
		if(e.key == 'w'){
			direction = [0, -1];
		}
		if(e.key == 'd'){
			direction = [1, 0];
		}
		if(e.key == 'a'){
			direction = [-1, 0];
		}
		if(e.key == 's'){
			direction = [0, 1];
		}
	}}
/>

<main>
	{#if !lost}
	<label>
		<input type=range bind:value={speed} min=0 max=15 />
	</label>
	<div>speed {speed} </div>
	<div class='wrapper' style='--gridSize: {gridSize}'>
		{#each squares as square}
			<Square type={square}></Square>
		{/each}
	</div>
	{:else}
	<div>
		You lost
	</div>
	<div>
		<button on:click={restart}>restart</button>
	</div>
	{/if}
</main>
<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		width:1200px;
		margin: 0 auto;
	}

	.wrapper {
		display: grid;
		grid-template-columns: repeat(var(--gridSize), 50px);
		grid-template-rows: repeat(var(--gridSize), 50px);
		grid-column-gap: 2px;
		grid-row-gap: 2px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>