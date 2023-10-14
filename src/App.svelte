<script>
	import { onMount, onDestroy } from 'svelte'; 
	import { readable, derived, get } from 'svelte/store';
	let isMenuVisible = false;

		function toggleMenu() {
			isMenuVisible = !isMenuVisible;
		}
	// State
	let questions = [];
	let selectedLanguage = "en";
	let randomQuestion;
	let gradientNumber = 1;
	let questionSet = "in"; 
	let showModal = false;

	const urlParams = readable(new URLSearchParams(window.location.search), set => {
		function updateParams() {
			set(new URLSearchParams(window.location.search));
		}
		window.addEventListener('popstate', updateParams);
		return () => window.removeEventListener('popstate', updateParams);
	});

	const currentQuestionId = derived(urlParams, $urlParams => $urlParams.get('id') || null);
	



	
	function close() {
	showModal = false;
	}	


	onMount(async () => {
		window.addEventListener('keydown', handleKeyDown);
		await loadQuestions();
	});

	onDestroy(() => {
		window.removeEventListener('keydown', handleKeyDown);
	});
	
	async function loadQuestions() {
		const response = await fetch(`/questions_${questionSet}.json`);
		questions = await response.json();
		randomQuestion = getRandomQuestion();
	}

	function getRandomQuestion() {
		return questions[Math.floor(Math.random() * questions.length)];
	}

	function switchQuestion() {
		randomQuestion = getRandomQuestion();
		gradientNumber = (gradientNumber % 10) + 1;
		updateUrl();
	}
	async function toggleSet() {
		questionSet = questionSet === "in" ? "out" : "in";
		await loadQuestions();
	}

	function handleKeyDown(event) {
		if (event.key === ' ') {
			switchQuestion();
		}
	}

	function toggleFullscreen() {
		let element = document.documentElement;
		if (!document.fullscreenElement) {
			element.requestFullscreen();
		} else {
			document.exitFullscreen();
		}
	}


	function updateUrl() {
		const id = questions.indexOf(randomQuestion);
		const params = new URLSearchParams();
		params.set('lang', selectedLanguage);
		params.set('gradient', gradientNumber);
		if (id !== -1) params.set('id', id);
		history.pushState({}, '', '?' + params.toString());
	}

	onMount(async () => {
		window.addEventListener('keydown', handleKeyDown);
		await loadQuestions();
		
		// Load state from URL
		const params = get(urlParams);
		if (params.has('lang')) selectedLanguage = params.get('lang');
		if (params.has('gradient')) gradientNumber = +params.get('gradient');
		if (params.has('id')) randomQuestion = questions[+params.get('id')];

		updateUrl();
	});

	
</script>

<style>

</style>

<div class={`full gradient_${gradientNumber}`}>
	





		<div class="side"> <span class="bar"><h1 style="color:#000000;">
					{questionSet === "in" ? "Check-In" : "Check-Out"}
				</h1>
			<button on:click={toggleSet} class="button">
				{questionSet === "in" ? "Out" : "In"}
			</button></span>

			<span class="bar"><select bind:value={selectedLanguage} class="dropdown">
				<option value="en" >English</option>
				<option value="de">German</option>
				<option value="it">Italian</option>
				<option value="fr">French</option>
			</select></span>

			<span class="bar" style="display: flex;
			flex-direction: column;
			align-items: center;"><button on:click={switchQuestion} class="button" style="margin-top:18px; margin-bottom:0px;">
				Shuffle
			</button> <p style="font-size:9px; margin:0px;">( Press Space )</p></span>

			<span class="bar">
			<button on:click={toggleFullscreen} class="button">
				Fullscreen
			</button>
			</span>
			<span class="bar">
			<button on:click={() => showModal = true}>Info</button>
			</span>
	
	</div>
	{#if randomQuestion}
		<div style="display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%;">
			<p class="question" >{randomQuestion[selectedLanguage]}</p>
		</div>
	{:else}
		<p>Loading...</p>
	{/if}
</div>

{#if showModal}
  <div class="modal">
    <div class="modal-content">
      <span class="close" on:click={close}>&times;</span>
      <p>
        Dummy text here. Lorem ipsum dolor sit, amet consectetur adipisicing elit.
        Voluptatem perspiciatis iste eos obcaecati accusantium cumque doloribus quaerat iusto aut.
      </p>
    </div>
  </div>
{/if}