<script>
	import Square from './Square.svelte';
	import { onMount, tick } from 'svelte/internal';
	export const GRID_SIZE = 15;
	export const TICK_DELAY_MAX = 300;
	
	let speed = 0;
	let lost = false;
	let direction = [-1, 0];
	let start = false;
	let score = 0;
	let highscore = 0;


	function toTuple(num) {
		return [num % GRID_SIZE, Math.floor(num / GRID_SIZE)];
	}

	function fromTuple(tuple) {
		let x = tuple[0];
		let y = tuple[1];
		return x + y * GRID_SIZE;
	}
	let squares = [];
	let food = new Set();
	food.add(fromTuple([3,3]));
	let snake = [fromTuple([10,10])];
	for (let i = 0; i < GRID_SIZE**2; i++) {
		squares = [...squares, 'empty'];
	}

	squares[snake[0]] = 'snake';
	for(let f of food){
		console.log(f);
		squares[f] = 'food';
	}

	function restart() {
		squares = [];
		for(let i = 0; i < GRID_SIZE**2; i++){
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
			if(coord < 0 || coord >= GRID_SIZE){
				lost = true;
				start = false;
				if (score > highscore) highscore = score;
				score = 0;
			}
		});
		if(snake.includes(fromTuple(head))){
			lost = true;
			start = false;
			if (score > highscore) highscore = score;
			score = 0;
		}
	}


	function playLoop() {

		setTimeout(() => {

			if (!start) return playLoop();

			let tail = snake[snake.length - 1];
			let head = move(snake[0]);
			if(food.has(head)){
				food.delete(head);
				let newFood = -1;
				do {
					newFood = Math.floor(Math.random() * GRID_SIZE**2);
				} while(snake.includes(newFood)) 
				console.log('newFood', newFood);
				food.add(newFood);
				squares[newFood] = 'food';
				snake = [head, ...snake];
				score++;
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
		if (e.key == ' ' && lost) {
			restart();
		}
		else if (e.key == ' ' && !start) {
			start = true;
		}
	}}
/>

<main>
	{#if !lost}
	<label>
		<input type=range bind:value={speed} min=0 max=15 />
		<button on:click={() => {start = true;}}>start</button>
	</label>
	<div>speed {speed} </div>
	<p>Score: <b>{score}</b>&#9; Highscore: <b>{highscore}</b></p>
	<div class="outer-wrapper">
		<div class='wrapper' style='--GRID_SIZE: {GRID_SIZE}'>
			{#each squares as square}
				<Square type={square}></Square>
			{/each}
		</div>
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
		grid-template-columns: repeat(var(--GRID_SIZE), 50px);
		grid-template-rows: repeat(var(--GRID_SIZE), 50px);
		grid-column-gap: 2px;
		grid-row-gap: 2px;
		margin: 0 auto;
	}

	.outer-wrapper {
		display: flex;
		width: 100%;
		justify-content: center;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>