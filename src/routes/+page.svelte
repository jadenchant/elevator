<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';

	let camera: THREE.PerspectiveCamera;
	let scene: THREE.Scene;
	let renderer: THREE.WebGLRenderer;
	let cube: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial>;
	let planeR: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial>;
	let planeL: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial>;
	let planeT: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshBasicMaterial>;
	let animationFrameId: number;

	const initScene = () => {
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

		renderer = new THREE.WebGLRenderer();
		renderer.setSize(window.innerWidth, window.innerHeight);
		document.body.appendChild(renderer.domElement);

		const boxgeo = new THREE.BoxGeometry(1, 2, 1);
		const boxmat = new THREE.MeshBasicMaterial({ color: 0xffffff });
		cube = new THREE.Mesh(boxgeo, boxmat);
		cube.position.y = 1;
		scene.add(cube);

		const planegeo = new THREE.PlaneGeometry(4, 2);
		const planemat = new THREE.MeshBasicMaterial({ color: 0xdddddd });
		planeR = new THREE.Mesh(planegeo, planemat);
		planeR.position.set(2.5, 1, 0.5);
		scene.add(planeR);

		planeL = new THREE.Mesh(planegeo, planemat);
		planeL.position.set(-2.5, 1, 0.5);
		scene.add(planeL);

		const planegeoT = new THREE.PlaneGeometry(9, 0.25);
		planeT = new THREE.Mesh(planegeoT, planemat);
		planeT.position.set(0, 2.125, 0.5);
		scene.add(planeT);

		// Temp Grid Helper
		const gridHelper = new THREE.GridHelper(10, 10);
		scene.add(gridHelper);

		camera.position.z = 5;
		camera.position.y = 1.2;
	};

	const renderScene = () => {
		renderer.render(scene, camera);
	};

	const animateScene = () => {
		animationFrameId = requestAnimationFrame(animateScene);

		// if (camera.position.z > 2) {
		// 	camera.position.z -= 0.005;
		// }

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
