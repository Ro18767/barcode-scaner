<script lang="ts">
	import { browser } from '$app/environment';
	import { clipboard } from '@skeletonlabs/skeleton';
	import { FileButton } from '@skeletonlabs/skeleton';

	let img: HTMLImageElement;
	let files: FileList;
	let file: File;
	let barcodeValues: string[] = [];
	let imgUrl: string;

	$: if (files) file = files[0];
	// $: console.log(file);
	$: if (file) imgUrl = URL.createObjectURL(file);
	// $: console.log(barcodeValues);

	let copied = false;
	let copiedIndex = 0;

	function onClickHandler(this: HTMLButtonElement, event: MouseEvent): void {
		copied = true;
		copiedIndex = +(this.dataset.index || 0);
		setTimeout(() => {
			copied = false;
		}, 1000);
	}

	async function onFinishLoading(event: Event) {
		barcodeValues = [];

		if (!browser) return;

		URL.revokeObjectURL(imgUrl);

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

<div class="container mx-auto p-4 space-y-4">
	{#if browser && 'BarcodeDetector' in window}
		<FileButton name="" bind:files accept="image/*">Select Image</FileButton>
	{:else}
		<h2>BarcodeDetector not supported</h2>
	{/if}

	{#if imgUrl}
		<div
			class="flex aspect-square text-surface-50 justify-center items-center overflow-hidden isolate w-48 max-w-full"
		>
			<img
				class="w-full h-full object-contain"
				src={imgUrl}
				alt="test"
				bind:this={img}
				on:load={onFinishLoading}
				on:error={onFinishLoading}
			/>
		</div>
	{/if}

	{#if barcodeValues.length}
		<div class="card">
			{#each barcodeValues as value, index}
				<section class="p-4 flex items-center gap-2">
					<input class="input p-2" type="text" {value} readonly />
					<button
						use:clipboard={value}
						class="btn variant-filled p-2"
						on:click={onClickHandler}
						data-index={index}
						disabled={copied && index == copiedIndex}
						>{copied && index == copiedIndex ? 'Copied 👍' : 'Copy'}</button
					>
				</section>
			{/each}
		</div>
	{/if}
</div>
