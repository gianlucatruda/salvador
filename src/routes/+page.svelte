<script lang="ts">
	let btnText = "Generate Image";
	let promptText = "A melting clock in the style of Dali.";
	let modelName = "dall-e-2";
	let imgSize: string;
	let uiDisabled = false;
	let imgURL = "";

	const makeImage = () => {
		console.log("Querying:", promptText);
		uiDisabled = true;
		btnText = "Painting...";
		makeRequest();
	};

	async function makeRequest() {
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
	<h1>Salvador</h1>
	<p>
		A bespoke interface to <a href="https://openai.com/index/dall-e-3/"
			>OpenAI's DALL.E</a
		>.
	</p>
	<div class="modelParams">
		<textarea
			id="prompt"
			rows="3"
			cols="40"
			wrap="soft"
			maxlength="999"
			bind:value={promptText}
			disabled={uiDisabled}
		></textarea>
		<form>
			<label for="modelSelector">Model:</label>
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
					<option value="dall-e-3">DALL.E 3</option>
				</optgroup>
			</select>
		</form>
		{#if modelName == "dall-e-2"}
			<form>
				<label for="sizeSelectorD2">Image Size:</label>
				<select
					name="sizeSelectorD2"
					id="sizeSelect"
					bind:value={imgSize}
					disabled={uiDisabled}
				>
					<option value="256x256">256 x 256 (fast and cheap)</option>
					<option value="512x512">512 x 512</option>
					<option value="1024x1024"
						>1024 x 1024 (slow and pricey)</option
					>
				</select>
			</form>
		{:else if modelName == "dall-e-3"}
			<form>
				<label for="sizeSelectorD3">Image Size:</label>
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
			<p>This should never happen...</p>
		{/if}
	</div>
	<button on:click={makeImage} disabled={uiDisabled}>{btnText}</button>
	<div class="results">
		{#if imgURL !== ""}
			<img src={imgURL} alt="result" />
		{/if}
	</div>
	<p>Prompt: {promptText}</p>
	<p>Model: {modelName}</p>
	<p>Size: {imgSize}</p>
	<footer>
		<p>
			Made with <a href="https://kit.svelte.dev/">Svelte</a> and love by
			<a href="https://gianluca.ai">Gianluca Truda</a>.
		</p>
	</footer>
</main>

<style>
	main {
		text-align: center;
		padding: 0.5em;
		max-width: 600px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 3em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
