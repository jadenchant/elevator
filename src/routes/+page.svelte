<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';

	const clock = new THREE.Clock();

	let camera: THREE.PerspectiveCamera;
	let scene: THREE.Scene;
	let renderer: THREE.WebGLRenderer;
	let doorR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let doorL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let wallR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let wallL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let wallT: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let lbfloor: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshStandardMaterial>;
	let elevWallR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallB: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallFR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallFL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallFT: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevFloor: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshStandardMaterial>;
	let animationFrameId: number;

	const initScene = () => {
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

		renderer = new THREE.WebGLRenderer();
		renderer.setSize(window.innerWidth, window.innerHeight);
		document.body.appendChild(renderer.domElement);

		const bricktexture = new THREE.TextureLoader().load('bricktexture.png');
		bricktexture.wrapS = THREE.RepeatWrapping;
		bricktexture.wrapT = THREE.RepeatWrapping;
		bricktexture.repeat.set(4, 3);

		const walnuttexture = new THREE.TextureLoader().load('walnut.png');
		walnuttexture.wrapS = THREE.RepeatWrapping;
		walnuttexture.wrapT = THREE.RepeatWrapping;
		walnuttexture.repeat.set(6, 3);

		const doorgeo = new THREE.BoxGeometry(0.5, 2, 0.05);
		const doormat = new THREE.MeshStandardMaterial({ color: 0xffffff });
		doorR = new THREE.Mesh(doorgeo, doormat);
		doorR.position.set(0.25, 1, -0.05);
		scene.add(doorR);

		doorL = new THREE.Mesh(doorgeo, doormat);
		doorL.position.set(-0.25, 1, -0.05);
		scene.add(doorL);

		const wallgeo = new THREE.BoxGeometry(6, 2, 0.05);
		const wallmat = new THREE.MeshStandardMaterial({ map: bricktexture });
		wallR = new THREE.Mesh(wallgeo, wallmat);
		wallR.position.set(3.5, 1, 0);
		scene.add(wallR);

		wallL = new THREE.Mesh(wallgeo, wallmat);
		wallL.position.set(-3.5, 1, 0);
		scene.add(wallL);

		const wallgeoT = new THREE.BoxGeometry(13, 2, 0.05);
		const wallmatT = new THREE.MeshStandardMaterial({ color: 0xdddddd });
		wallT = new THREE.Mesh(wallgeoT, wallmatT);
		wallT.position.set(0, 3, 0);
		scene.add(wallT);

		const lbfloorgeo = new THREE.PlaneGeometry(17, 8);
		const lbfloormat = new THREE.MeshStandardMaterial({ map: walnuttexture });
		lbfloor = new THREE.Mesh(lbfloorgeo, lbfloormat);
		lbfloor.position.set(0, 0, 3.95);
		lbfloor.rotation.x = -Math.PI / 2;
		scene.add(lbfloor);

		const elevWallgeo = new THREE.BoxGeometry(2.5, 2.4, 0.05);
		const elevwallmat = new THREE.MeshStandardMaterial({ color: 0xbbbbbb });
		elevWallR = new THREE.Mesh(elevWallgeo, elevwallmat);
		elevWallR.position.set(1.5, 1.2, -1.25);
		elevWallR.rotation.y = Math.PI / 2;
		scene.add(elevWallR);

		elevWallL = new THREE.Mesh(elevWallgeo, elevwallmat);
		elevWallL.position.set(-1.5, 1.2, -1.25);
		elevWallL.rotation.y = Math.PI / 2;
		scene.add(elevWallL);

		const elevWallgeoB = new THREE.BoxGeometry(4, 2.4, 0.05);
		elevWallB = new THREE.Mesh(elevWallgeoB, elevwallmat);
		elevWallB.position.set(0, 1.2, -2.5);
		scene.add(elevWallB);

		const elevWallgeoF = new THREE.BoxGeometry(1, 2.4, 0.05);
		elevWallFL = new THREE.Mesh(elevWallgeoF, elevwallmat);
		elevWallFL.position.set(1, 1.2, -0.1);
		scene.add(elevWallFL);

		elevWallFR = new THREE.Mesh(elevWallgeoF, elevwallmat);
		elevWallFR.position.set(-1, 1.2, -0.1);
		scene.add(elevWallFR);

		const elevWallgeoFT = new THREE.BoxGeometry(1, 0.4, 0.05);
		elevWallFT = new THREE.Mesh(elevWallgeoFT, elevwallmat);
		elevWallFT.position.set(0, 2.2, -0.1);
		scene.add(elevWallFT);

		const elevFloorgeo = new THREE.PlaneGeometry(3, 3);
		const elevFloormat = new THREE.MeshStandardMaterial({ color: 0xdddddd });
		elevFloor = new THREE.Mesh(elevFloorgeo, elevFloormat);
		elevFloor.position.set(0, 0, -1.55);
		elevFloor.rotation.x = -Math.PI / 2;
		scene.add(elevFloor);

		const al = new THREE.AmbientLight(0xffffff, 0.25);
		scene.add(al);

		const lpl1 = new THREE.PointLight(0xffffff, 12, 10, 1.2);
		lpl1.position.set(4, 6, 4);
		const lpl1Helper = new THREE.PointLightHelper(lpl1, 0.5);
		scene.add(lpl1, lpl1Helper);

		const lpl2 = new THREE.PointLight(0xffffff, 12, 10, 1.2);
		lpl2.position.set(-4, 6, 4);
		const lpl2Helper = new THREE.PointLightHelper(lpl2, 0.5);
		scene.add(lpl2, lpl2Helper);

		// Temp Grid Helper
		// const gridHelper = new THREE.GridHelper(15, 15);
		// scene.add(gridHelper);

		camera.position.z = 3.5;
		camera.position.y = 1.3;

		// Temp inside elevator
		// camera.position.z = -2;
		// camera.rotation.y = Math.PI;

		// Temp bird view
		// camera.position.z = 0;
		// camera.position.y = 6;
		// camera.rotation.x = -Math.PI / 2;
	};

	const renderScene = () => {
		renderer.render(scene, camera);
	};

	const animateScene = () => {
		animationFrameId = requestAnimationFrame(animateScene);

		const delta = clock.getDelta();
		const sec = clock.getElapsedTime();

		// if (sec > 0.6) {
		// 	if (camera.position.z > -1.2) {
		// 		camera.position.z -= 0.01;
		// 	} else if (camera.rotation.y < Math.PI) {
		// 		camera.rotation.y += 0.02;
		// 		camera.position.x -= 0.005;
		// 	}
		// }

		// if (sec < 8 && sec > 2 && doorL.position.x > -0.75) {
		// 	doorL.position.x -= 0.005;
		// 	doorR.position.x += 0.005;
		// } else if (sec > 12 && doorL.position.x < -0.25) {
		// 	doorL.position.x += 0.005;
		// 	doorR.position.x -= 0.005;
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
