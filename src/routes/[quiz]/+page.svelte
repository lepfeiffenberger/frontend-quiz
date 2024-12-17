<script>
	import { page } from '$app/stores';
	import ListItem from '$lib/components/listItem.svelte';
	let { data } = $props();
	let currentQuestion = $state(0);
	let selectedAnswer = $state(' ');
	let score = $state(0);
	let submitted = $state(false);

	function handleSubmit() {
		data.questions[currentQuestion].selectedAnswer = selectedAnswer;
		if (selectedAnswer === data.questions[currentQuestion].answer) {
			score += 1;
		}
		submitted = true;
	}

	function nextQuestion() {
		currentQuestion++;
		submitted = false;
		progress.update((n) => {
			if (n + 0.1 > 1) {
				return 1;
			} else {
				return n + 0.1;
			}
		});
	}

	import { writable } from 'svelte/store';

	const progress = writable(0.1);

	function saveData() {
		localStorage.setItem('score', score);
		localStorage.setItem('currentQuestion', currentQuestion);
	}

	$effect(() => {
		score = localStorage.getItem('score');
		currentQuestion = localStorage.getItem('currentQuestion');
	});
</script>

<progress value={$progress}></progress>
{#if currentQuestion < data.questions.length}
	{#each data.questions as question, index (question.question)}
		{#if index === currentQuestion}
			<p>Question {index + 1} of {data.questions.length}</p>
			<br />
			<h2>{question.question}</h2>
			<br />
			{#each question.options as option, index (option)}
				{#key submitted}
					<ListItem
						title={option}
						{index}
						correctAnswer={question.selectedAnswer && question.answer === option}
						incorrectlySelected={question.selectedAnswer === option && question.answer !== option}
						focusAction={() => (selectedAnswer = option)}
						disabled={submitted}
					></ListItem>
				{/key}
			{/each}
		{/if}
	{/each}

	<br />

	{#if !submitted}
		<button class="btn btn-primary" disabled={!selectedAnswer} onclick={handleSubmit}>Submit</button
		>
	{:else}
		<button class="btn btn-primary" onclick={nextQuestion}>Next Question</button>
	{/if}

	<br />
	<br />
	<h1>Punkte: {score}</h1>
{:else}
	<h2>Game Over!</h2>
	<p>You scored {score} out of {data.questions.length} points.</p>
	<button class="btn btn-primary" onclick={window.location.reload()}>Play again!</button>
{/if}

<style>
	progress {
		display: block;
		border-top-left-radius: 10px;
		border-bottom-left-radius: 10px;
		border-top-right-radius: 10px;
		border-bottom-right-radius: 10px;
		width: 25%;
		background-color: white;
		border-color: black;
		color: blueviolet;
	}
</style>
