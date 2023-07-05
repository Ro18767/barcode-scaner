<script lang="ts">
	import { browser } from '$app/environment';

	let img: HTMLImageElement;
	let files: FileList;
	let file: File;
	let barcodeValues: string[] = [];
	let imgUrl: string;

	$: if (files) file = files[0];
	$: if (file && img) img.src = URL.createObjectURL(file);

	async function onFinishLoading(event: Event) {
		barcodeValues = [];

		if (!browser) return;

		URL.revokeObjectURL(img.src);

		if (event.type !== 'load') return;

		if (!('BarcodeDetector' in window)) return;
		if (typeof window.BarcodeDetector !== 'function') return;

		// @ts-ignore
		const barcodeDetector = new window.BarcodeDetector();

		barcodeValues = await barcodeDetector
			.detect(img)
			.catch((error: unknown) => {
				console.error(error);
				return [];
			})
			.then((barcodes: { rawValue: string }[]) => {
				return barcodes.map((barcode) => barcode.rawValue);
			});
	}
</script>

{#if browser && 'BarcodeDetector' in window}
	<input type="file" bind:files accept="image/*" />
{:else}
	<h2>BarcodeDetector not supported</h2>
{/if}

{#if imgUrl}
	<img
		src={imgUrl}
		alt="test"
		bind:this={img}
		on:load={onFinishLoading}
		on:error={onFinishLoading}
	/>
{/if}

{#each barcodeValues as value}
	<input type="text" {value} />
{/each}

<style>
	input {
		display: block;
	}
	img {
		display: block;
	}
</style>
