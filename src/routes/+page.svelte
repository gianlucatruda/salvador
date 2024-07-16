<script lang="ts">
	import { onMount } from "svelte";

	let btnText = "Generate Image";
	let promptText = "";
	let modelName = "dall-e-2";
	let imgSize = "256x256";
	let uiDisabled = false;
	let imgURL = "";
	let lastPrompt: string | null;
	onMount(() => {
		if (localStorage.getItem("lastPrompt")) {
			lastPrompt = localStorage.getItem("lastPrompt");
		}
	});

	const loadPreviousPrompt = () => {
		if (!lastPrompt) {
			console.error("No last prompt specified.");
		}
		promptText = lastPrompt;
	};

	const makeImage = () => {
		const query = `${modelName} (${imgSize}): "${promptText}"`;
		console.log("Query:", query);
		uiDisabled = true;
		btnText = "Generating...";
		makeRequest();
	};

	async function makeRequest() {
		localStorage.setItem("lastPrompt", promptText);
		const bodyData = {
			model: modelName,
			prompt: promptText,
			n: 1,
			size: imgSize,
			// TODO quality (for Dalle3)
			// TODO style (for Dalle3)
		};
		const response = await fetch("/api/generate", {
			method: "POST",
			headers: {
				"Content-Type": "application/json",
			},
			body: JSON.stringify(bodyData),
		});

		const result = await response.json();
		console.log(result);
		imgURL = result.data[0].url;
	}
</script>

<main>
	<link
		rel="stylesheet"
		type="text/css"
		href="//fonts.googleapis.com/css?family=Dancing+Script"
	/>
	<h1>Salvador</h1>
	<p>
		A bespoke interface to <a href="https://openai.com/index/dall-e-3/"
			>OpenAI's DALL.E</a
		>.
	</p>
	<div class="modelParams">
		<form>
			<label for="modelSelector">Model</label>
			<select
				name="modelSelector"
				id="modelSelect"
				bind:value={modelName}
				disabled={uiDisabled}
			>
				<optgroup label="Older, cheaper, wackier">
					<option value="dall-e-2">DALL.E 2</option>
				</optgroup>
				<optgroup label="Newer, higher-res, slower">
					<option value="dall-e-3" disabled={true}
						>DALL.E 3 (coming soon!)</option
					>
				</optgroup>
			</select>
		</form>
		{#if modelName == "dall-e-2"}
			<form>
				<label for="sizeSelectorD2">Image Size</label>
				<select
					name="sizeSelectorD2"
					id="sizeSelect"
					bind:value={imgSize}
					disabled={uiDisabled}
				>
					<option value="256x256"
						>256 x 256 (fast and reliable)</option
					>
					<option value="512x512">512 x 512</option>
					<option value="1024x1024"
						>1024 x 1024 (slow, pricey, brittle)</option
					>
				</select>
			</form>
		{:else if modelName == "dall-e-3"}
			<form>
				<label for="sizeSelectorD3">Image Size</label>
				<select
					name="sizeSelectorD3"
					id="sizeSelect"
					bind:value={imgSize}
					disabled={uiDisabled}
				>
					<option value="1024x1024" selected={modelName == "dall-e-3"}
						>1024 x 1024</option
					>
					<option value="1792x1024">1792 x 1024 (landscape)</option>
					<option value="1024x1792">1024 x 1792 (portrait)</option>
				</select>
			</form>
		{:else}
			<p>This should never happen. Model name is invalid: {modelName}</p>
		{/if}
		<textarea
			id="prompt"
			rows="3"
			cols="40"
			wrap="soft"
			maxlength="999"
			minlength="10"
			placeholder="A painting of a melting clock in the style of Dali..."
			bind:value={promptText}
			disabled={uiDisabled}
		></textarea>
		{#if lastPrompt && !promptText}
			<button
				class="loadPrevButton"
				on:click={loadPreviousPrompt}
				disabled={promptText == lastPrompt}>Load previous prompt</button
			>
		{:else if !imgURL}
			<button
				class="clearPromptButton"
				on:click={() => {
					promptText = "";
				}}
				disabled={!promptText || uiDisabled}>Clear prompt</button
			>
		{/if}
	</div>
	{#if !imgURL}
		<button on:click={makeImage} disabled={uiDisabled || promptText == ""}
			>{btnText}</button
		>
	{/if}
	<div class="results">
		{#if imgURL !== ""}
			<img src={imgURL} alt="Generated AI artwork" />
			<p>
				On your phone, tap and hold the image > <b>Save to Photos</b>.
				On your computer, right click the image >
				<b>Save Image As...</b>.
			</p>
		{:else if uiDisabled}
			<div class="spinner"></div>
		{/if}
	</div>
	{#if uiDisabled}
		<p>Query: {`${modelName} (${imgSize}): "${promptText}"`}</p>
	{/if}

	{#if imgURL}
		<button class="resetButton" on:click={() => location.reload()}
			>Reset Everything</button
		>
	{/if}

	<footer>
		<p>
			Made with <a href="https://kit.svelte.dev/">Svelte</a> and love by
			<a href="https://gianluca.ai">Gianluca Truda</a>. Happy Birthday,
			mom!
		</p>
	</footer>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 500px;
		margin: 0 auto; /* Needed for centering */
		background: white;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
		border-radius: 20px;
	}
	.modelParams {
		display: inline-block;
	}

	h1 {
		/* color: #007aff; iOS blue */
		color: #000000;
		text-transform: none;
		font-size: 5em;
		font-weight: 900;
		font-family:
			Dancing Script,
			Cambria,
			Cochin,
			Georgia,
			Times,
			"Times New Roman",
			serif;
	}

	p,
	a,
	label {
		font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
			"Helvetica Neue", Arial, sans-serif;
		color: #666;
	}

	textarea,
	select,
	button {
		font-size: 18px;
		padding: 10px;
		border: 1px solid #ccc;
		border-radius: 10px;
		width: 90%;
		margin: 10px 0;
	}

	textarea {
		height: 100px; /* Better touch area */
	}

	button {
		background-color: #007aff;
		color: white;
		text-transform: uppercase;
		letter-spacing: 1px;
		transition: background-color 0.3s;
	}

	button:disabled {
		opacity: 0.5;
	}

	.resetButton {
		background-color: orange;
	}

	.loadPrevButton,
	.clearPromptButton {
		background-color: grey;
	}

	@media (min-width: 640px) {
		main {
			padding: 2em;
			margin-top: 5vh;
		}

		h1 {
			font-size: 4em;
		}
	}

	.results img {
		width: 70%;
		max-width: 500px;
		height: auto;
		border-radius: 10px;
	}

	@keyframes spinner {
		to {
			transform: rotate(360deg);
		}
	}
	.spinner {
		height: 40px;
	}

	.spinner:before {
		content: "";
		box-sizing: border-box;
		position: absolute;
		width: 30px;
		height: 30px;
		margin-top: 5px;
		margin-left: -15px;
		border-radius: 50%;
		border: 4px solid #ccc;
		border-top-color: #000;
		animation: spinner 0.5s linear infinite;
	}
</style>
