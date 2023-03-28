<script lang="ts">
	import Scene from '$lib/Scene.svelte';
	import { Canvas, type ThrelteContext } from '@threlte/core';

	let three: ThrelteContext;
</script>

<Canvas shadows={true} bind:ctx={three} linear flat>
	<Scene />
</Canvas>
