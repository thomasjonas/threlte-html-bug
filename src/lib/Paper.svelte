<script lang="ts">
	import { InteractiveObject, T, useThrelte } from '@threlte/core';
	import { HTML } from '@threlte/extras';
	import { cubicInOut } from 'svelte/easing';
	import { tweened } from 'svelte/motion';
	import {
		BackSide,
		Camera,
		Color,
		DoubleSide,
		FrontSide,
		PerspectiveCamera,
		type ColorRepresentation,
		type Mesh
	} from 'three';
	import { degToRad, randFloat } from 'three/src/math/MathUtils';
	import { PAPER_THICKNESS } from '../config';
	import Content from './Content.svelte';

	export let size: [number, number];
	export let index: number;
	export let color: ColorRepresentation;

	$: isPortrait = size[1] > size[0];

	let mesh: Mesh;

	const ANIM_CONFIG = {
		easing: cubicInOut,
		duration: 500
	};

	function getRandomPosition() {
		return { x: randFloat(-4, 4), y: randFloat(-4, 4), z: index * PAPER_THICKNESS };
	}

	const { camera, renderer } = useThrelte();

	function isPerspectiveCamera(camera: Camera): camera is PerspectiveCamera {
		return (camera as PerspectiveCamera).isPerspectiveCamera === true;
	}

	function getPositionZ() {
		if (!isPerspectiveCamera($camera)) return;

		// if paper is landscape, we rotate paper to portrait position (as portrait is preferable for reading/viewing/phones)
		// that's why we have to switch width and height here
		const w = isPortrait ? size[0] : size[1];
		const h = isPortrait ? size[1] : size[0];
		const paperAspect = w / h;

		const max = Math.max(
			renderer?.domElement.clientHeight ?? 0,
			renderer?.domElement.clientWidth ?? 0
		);
		const margin = max < 800 ? 1.1 : 1.25;

		let z = 0;
		if (paperAspect <= $camera.aspect) {
			// fit height
			const d = (margin * h) / (2 * Math.tan(degToRad($camera.fov / 2)));
			z = $camera.position.z - d - 0.01;
		} else {
			// fit width
			const d = (margin * w) / ($camera.aspect * 2 * Math.tan(degToRad($camera.fov / 2)));
			z = $camera.position.z - d - 0.01;
		}

		return z;
	}

	const pos = tweened(getRandomPosition(), ANIM_CONFIG);
	const rotation = tweened(
		{
			z: Math.random() * Math.PI,
			x: 0
		},
		ANIM_CONFIG
	);
	const textOpacity = tweened(0, ANIM_CONFIG);

	let selected = false;

	$: {
		if (selected) {
			pos.set({ x: 0, y: 0, z: getPositionZ() ?? 6 });
			rotation.set({ z: isPortrait ? 0 : Math.PI / 2, x: Math.PI });
			textOpacity.set(1);
		} else {
			pos.set(getRandomPosition());
			rotation.set({ z: Math.random() * Math.PI, x: 0 });
			textOpacity.set(0);
		}
	}

	const toggle = () => {
		selected = !selected;
	};
</script>

<T.Group position={[$pos.x, $pos.y, $pos.z]} rotation.z={$rotation.z} rotation.x={$rotation.x}>
	<T.Mesh bind:ref={mesh} castShadow={true} receiveShadow let:ref>
		<InteractiveObject object={ref} interactive on:click={toggle} />

		<T.BoxGeometry args={[size[0], size[1], PAPER_THICKNESS]} />
		<T.MeshStandardMaterial side={FrontSide} {color} />
	</T.Mesh>

	<T.Mesh castShadow={false} receiveShadow={true} position.z={-0.004}>
		<T.PlaneGeometry args={size} />
		<T.MeshStandardMaterial shadowSide={BackSide} side={DoubleSide} color="#aaa" />
		{#if $textOpacity > 0}
			<HTML
				transform
				distanceFactor={0.6667}
				rotation={{ x: Math.PI, y: 0, z: isPortrait ? 0 : Math.PI / 2 }}
			>
				<Content {size} opacity={$textOpacity} on:click={toggle} />
			</HTML>
		{/if}
	</T.Mesh>
</T.Group>
