<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';

	const clock = new THREE.Clock();

	let camera: THREE.PerspectiveCamera;
	let scene: THREE.Scene;
	let renderer: THREE.WebGLRenderer;
	let doorR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial>;
	let doorL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial>;
	let wallR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial>;
	let wallL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial>;
	let planeT: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial>;
	let animationFrameId: number;

	const initScene = () => {
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

		renderer = new THREE.WebGLRenderer();
		renderer.setSize(window.innerWidth, window.innerHeight);
		document.body.appendChild(renderer.domElement);

		const doorgeo = new THREE.BoxGeometry(0.5, 2, 0.05);
		const doormat = new THREE.MeshBasicMaterial({ color: 0xffffff });
		doorR = new THREE.Mesh(doorgeo, doormat);
		doorR.position.set(0.25, 1, -0.05);
		scene.add(doorR);

		doorL = new THREE.Mesh(doorgeo, doormat);
		doorL.position.set(-0.25, 1, -0.05);
		scene.add(doorL);

		const wallgeo = new THREE.BoxGeometry(6, 2, 0.05);
		const wallmat = new THREE.MeshBasicMaterial({ color: 0xdddddd });
		wallR = new THREE.Mesh(wallgeo, wallmat);
		wallR.position.set(3.5, 1, 0);
		scene.add(wallR);

		wallL = new THREE.Mesh(wallgeo, wallmat);
		wallL.position.set(-3.5, 1, 0);
		scene.add(wallL);

		const wallgeoT = new THREE.BoxGeometry(13, 0.4, 0.05);
		planeT = new THREE.Mesh(wallgeoT, wallmat);
		planeT.position.set(0, 2.2, 0);
		scene.add(planeT);

		// Temp Grid Helper
		const gridHelper = new THREE.GridHelper(15, 15);
		scene.add(gridHelper);

		camera.position.z = 4;
		camera.position.y = 1.35;
	};

	const renderScene = () => {
		renderer.render(scene, camera);
	};

	const animateScene = () => {
		animationFrameId = requestAnimationFrame(animateScene);

		const delta = clock.getDelta();
		const sec = clock.getElapsedTime();

		if (camera.position.z > -1) {
			camera.position.z -= 0.01;
		} else if (camera.rotation.y < 3.14) {
			camera.rotation.y += 0.01;
		}

		if (sec < 8 && sec > 2 && doorL.position.x > -0.75) {
			doorL.position.x -= 0.005;
			doorR.position.x += 0.005;
		} else if (sec > 8 && doorL.position.x < -0.25) {
			doorL.position.x += 0.005;
			doorR.position.x -= 0.005;
		}

		renderScene();
	};

	const onWindowResize = () => {
		camera.aspect = window.innerWidth / window.innerHeight;
		camera.updateProjectionMatrix();
		renderer.setSize(window.innerWidth, window.innerHeight);
	};

	onMount(() => {
		if (browser) {
			initScene();
			animateScene();
			window.addEventListener('resize', onWindowResize);
		}
	});

	onDestroy(() => {
		if (browser) {
			cancelAnimationFrame(animationFrameId);
			window.removeEventListener('resize', onWindowResize);
			document.body.removeChild(renderer.domElement);
		}
	});
</script>

<svelte:head>
	<title>Elevator</title>
</svelte:head>
