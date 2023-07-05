<script lang="ts">
	import { browser } from '$app/environment';
	import imgUrl from '$lib/assets/test.jpg?url';
	import { onMount } from 'svelte';
	let img: HTMLImageElement;
</script>

{#if browser && 'BarcodeDetector' in window}
	<img
		src={imgUrl}
		alt="test"
		on:load={async () => {
			if (!browser) return;
			console.log('BarcodeDetector', BarcodeDetector);
			BarcodeDetector.getSupportedFormats().then((supportedFormats) => {
				supportedFormats.forEach((format) => console.log(format));
			});
			const barcodeDetector = new BarcodeDetector();
			const barcodes = await barcodeDetector.detect(videoEl).catch((error) => {
				console.error(error);
				return [];
			});
			console.log('barcodes', barcodes);
		}}
	/>
{:else}
	<h2>BarcodeDetector not supported</h2>
{/if}
