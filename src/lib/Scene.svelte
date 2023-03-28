<script lang="ts">
	import { T, Three } from '@threlte/core';
	import { DirectionalLight } from 'three';
	import Paper from './Paper.svelte';

	let light = new DirectionalLight();
	light.shadow.mapSize.width = 2048;
	light.shadow.mapSize.height = 2048;
	light.shadow.camera.near = 0.1;
	light.shadow.camera.far = 100;
	light.shadow.camera.left = -20;
	light.shadow.camera.right = 20;
	light.shadow.camera.top = 10;
	light.shadow.camera.bottom = -10;
	light.shadow.bias = -0.0001;
	light.position.set(-3, 5, 10);
	light.castShadow = true;
	light.intensity = 0.5;
</script>

<T.PerspectiveCamera
	lookAt={{ x: 0, y: 0, z: 0 }}
	makeDefault
	position={[0, 0, 10]}
	fov={48}
	near={0.1}
	far={100}
/>

<Three type={light} />
<T.AmbientLight position={[0, 10, 20]} intensity={0.8} />

<Paper size={[1.5, 1.7]} index={0} color="blue" />
<Paper size={[1.5, 2]} index={1} color="red" />

<T.Mesh receiveShadow position={[0, 0, -0.15]}>
	<T.PlaneGeometry args={[100, 100]} />
	<T.MeshStandardMaterial color="white" />
</T.Mesh>
