<script lang="ts">
	import { browser } from '$app/environment';
	import imgUrl from '$lib/assets/test.jpg?url';
	import { onMount } from 'svelte';
	let img: HTMLImageElement;
	async function load() {
		if (!browser) return;
		if (!('BarcodeDetector' in window)) return;
		if (typeof window.BarcodeDetector !== 'function') return;
		
		console.log('BarcodeDetector', window.BarcodeDetector);

		// @ts-ignore
		window.BarcodeDetector.getSupportedFormats().then((supportedFormats) => {
			// @ts-ignore
			supportedFormats.forEach((format) => console.log(format));
		});
		// @ts-ignore
		const barcodeDetector = new window.BarcodeDetector();
		const barcodes = await barcodeDetector.detect(img).catch((error:unknown) => {
			console.error(error);
			return [];
		});
		console.log('barcodes', barcodes);
	}
</script>

{#if browser && 'BarcodeDetector' in window}
	<img src={imgUrl} alt="test" bind:this={img} on:load={load} />
{:else}
	<h2>BarcodeDetector not supported</h2>
{/if}
