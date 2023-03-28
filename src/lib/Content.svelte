<script lang="ts">
	import { createEventDispatcher } from 'svelte';

	export let size: [number, number];
	export let opacity: number;

	$: ratio = size[0] / size[1];
	$: isPortrait = size[0] < size[1];
	const SIZE = 1000;

	const dispatch = createEventDispatcher();
</script>

<div
	class="content"
	style="width: {isPortrait
		? ratio * SIZE
		: (1 / ratio) * SIZE}px; opacity: {opacity}; height: {SIZE}px;"
>
	<button type="button" on:click={() => dispatch('click')}>Close</button>
	<h1>Content</h1>
	<p>Content goes here...</p>
</div>

<style>
	.content {
		background: rgba(255, 0, 0, 0.4);
	}
</style>
