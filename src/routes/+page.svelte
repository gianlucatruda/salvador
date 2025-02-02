<script lang="ts">
	import { onMount } from "svelte";

	let btnText = "Generate Image";
	let promptText = "";
	let modelName = "dall-e-2";
	let imgSize = "256x256";
	let uiDisabled = false;
	let imgURL = "";
	let lastPrompt: string | null;
	let ownApiKey: string = null;
	let apiKey: string = null;

	onMount(() => {
		if (localStorage.getItem("lastPrompt")) {
			lastPrompt = localStorage.getItem("lastPrompt");
		}
		if (localStorage.getItem("apiKey")) {
			apiKey = localStorage.getItem("apiKey") || "";
		}
	});

	const loadPreviousPrompt = () => {
		if (!lastPrompt) {
			console.error("No last prompt specified.");
		}
		promptText = lastPrompt || "";
	};

	const saveApiKey = () => {
		if (ownApiKey.length > 0) {
			apiKey = ownApiKey;
			localStorage.setItem("apiKey", apiKey);
			alert("API key saved!");
		}
	};
	const clearApiKey = () => {

		// Confirm if sure
		if (confirm("Are you sure you want to delete your API key? You will need to re-enter it manually.")) {
			localStorage.removeItem("apiKey");
			apiKey = null;
			ownApiKey = null;
			alert("API key destroyed!");
		}
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
		};

		// If the user has set their own OpenAI API key, straight request
		let requestURL = "/api/generate";
		if (apiKey != null && apiKey != "") {
			requestURL = "https://api.openai.com/v1/images/generations";
		}
		console.log("Attempting request to: ", requestURL);

		const response = await fetch(requestURL, {
			method: "POST",
			headers: {
				"Content-Type": "application/json",
				"Authorization": `Bearer ${apiKey}`,
			},
			body: JSON.stringify(bodyData),
		});

		if (!response.ok) {
			console.warn(response.status, response.statusText);
			const errorResult = await response.json();
			console.error(errorResult.error.message);
			alert(`Error: ${errorResult.error.message}`);
			uiDisabled = false;
			btnText = "Generate Image";
			return;
		}

		const result = await response.json();
		console.log(result);
		if (result.error) {
			console.error(result.error);
			alert(JSON.stringify(result.error));
			uiDisabled = false;
			btnText = "Generate Image";
		} else {
			imgURL = result.data[0].url;
			uiDisabled = false;
			btnText = "Generate Image";
		}
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
				id="modelSelector"
				bind:value={modelName}
				disabled={uiDisabled}
			>
				<optgroup label="Older, cheaper, wackier">
					<option value="dall-e-2">DALL.E 2</option>
				</optgroup>
				<optgroup label="Newer, higher-res, slower">
					<option value="dall-e-3" disabled={!apiKey}
						>DALL.E 3 (use your own API key)</option
					>
				</optgroup>
			</select>
		</form>
		{#if modelName == "dall-e-2"}
			<form>
				<label for="sizeSelectorD2">Image Size</label>
				<select
					name="sizeSelectorD2"
					id="sizeSelectorD2"
					bind:value={imgSize}
					disabled={uiDisabled}
				>
					<option value="256x256"
						>256 x 256 (fast and reliable)</option
					>
					<option value="512x512">512 x 512</option>
					<option value="1024x1024" disabled={!apiKey}
						>1024 x 1024 (slow, pricey, brittle)</option
					>
				</select>
			</form>
		{:else if modelName == "dall-e-3"}
			<form>
				<label for="sizeSelectorD3">Image Size</label>
				<select
					name="sizeSelectorD3"
					id="sizeSelectorD3"
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
	</div>
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
	{#if !imgURL}
		<button on:click={makeImage} disabled={uiDisabled || !promptText}
			>{btnText}</button
		>
	{/if}
	<div class="results">
		{#if imgURL}
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
	<div class="api-key-section">
	{#if !apiKey}
		<p for="apiKey">Add your own <a href="https://platform.openai.com/api-keys">OpenAI API key</a> for unlimited generations, higher resolution, and most powerful models. This is only stored on your device.</p>
		<label for="apiKey">Your OpenAI API key</label>
		<input
			type="password"
			id="apiKey"
			bind:value={ownApiKey}
			placeholder="sk-..."
			disabled={uiDisabled}
		/>
		<button on:click={saveApiKey} disabled={uiDisabled || !ownApiKey}>
			Save API Key
		</button>
	{/if}
	{#if apiKey}
		<button class="resetButton" on:click={clearApiKey} disabled={uiDisabled}>
			Forget my API key
		</button>
	{/if}
	</div>

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
