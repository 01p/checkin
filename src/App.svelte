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
	let lastTouchTime = 0;

function handleDoubleClickTap() {
	const now = new Date().getTime();
	const timesince = now - lastTouchTime;

	if ((timesince < 300) && (timesince > 0)) {
		switchQuestion();
	}
	
	lastTouchTime = now;
}

onMount(async () => {
	window.addEventListener('keydown', handleKeyDown);
	window.addEventListener('touchend', handleDoubleClickTap);
	await loadQuestions();
});

onDestroy(() => {
	window.removeEventListener('keydown', handleKeyDown);
	window.removeEventListener('touchend', handleDoubleClickTap);
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
		/* Show question, gradient and language in url
		const id = questions.indexOf(randomQuestion);
		const params = new URLSearchParams();
		params.set('lang', selectedLanguage);
		params.set('gradient', gradientNumber);
		if (id !== -1) params.set('id', id);
		history.pushState({}, '', '?' + params.toString());*/
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
	

		<div class="side-mobil md:hidden" style="margin-left:10%;" > 
			<span class="bar">
			<button on:click={toggleSet} class="button-mobile" >
				{questionSet === "in" ? "O" : "I"}
			</button></span>

			<span class="bar" ><select bind:value={selectedLanguage} class="dropdown button-mobile">
				<option value="en">EN</option>
				<option value="de">DE</option>
				<option value="it">IT</option>
				<option value="fr">FR</option>
			</select></span>

			<span class="bar" style="display: flex;
			flex-direction: column;
			align-items: center;"><button on:click={switchQuestion} class="button-mobile" >
				S
			</button></span> <p style="font-size:9px; margin:0px;  writing-mode: vertical-rl;
			text-orientation: upright;"> Double Tap</p>

			<span class="bar">
			<button on:click={toggleFullscreen} class="button-mobile">
				F
			</button>
			</span>
			<span class="bar">
			<button on:click={() => showModal = true} class="button-mobile">I</button>
			</span>
	
	</div>

	<div class="side desktop sm:hidden md:flex" style="display:none;"> <span class="bar"><h1 style="color:#000000;">
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
      <span class="close" on:click={close}  on:keydown={close}>&times;</span>
     <div style="font-size: 28px; margin-top:4vv; max-width: 70%;">

		<p >
			Hey there , </p>
            <p>
            it's the year of efficiency! Checking-in and checking-out with your team by answering a few question together will help everyone to work more productive and aligned to  power through to the best results. 
        </p>
		<h2>
			Share remotely 
		</h2>
			If you work remotely,just share the link (in Slack, Notion,..), and every one will have the same question.
		<p>
			<h2>
				Create you own set of questions
			</h2>
	By default 

		
		 
	 </div> 
    </div>
  </div>
{/if}