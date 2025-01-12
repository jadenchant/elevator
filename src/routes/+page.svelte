<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
	import { Reflector } from '../lib/Reflector';

	let dev = false;

	const clock = new THREE.Clock();

	let camera: THREE.PerspectiveCamera;
	let scene: THREE.Scene;
	let renderer: THREE.WebGLRenderer;
	let controls: OrbitControls;
	let doorRF: Reflector;
	let doorLF: Reflector;
	let doorRB: Reflector;
	let doorLB: Reflector;
	let wallR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallEnter: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallExit: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallEnterT: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallExitT: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallB: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallT: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let wallTB: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let lbFloor: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshPhysicalMaterial>;
	let lbCeil: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshStandardMaterial>;
	let elevWallR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallB: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallFR: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallFL: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevWallFT: THREE.Mesh<THREE.BoxGeometry, THREE.MeshStandardMaterial>;
	let elevFloor: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshStandardMaterial>;
	let elevCeil: THREE.Mesh<THREE.PlaneGeometry, THREE.MeshStandardMaterial>;
	let elevPanel1: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let elevPanel2: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let elevPanel3: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let elevPanel4: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let elevPanel5: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let elevPanel6: THREE.Mesh<THREE.BoxGeometry, THREE.MeshPhysicalMaterial>;
	let lpl1: THREE.PointLight;
	let lpl2: THREE.PointLight;
	let lpl3: THREE.PointLight;
	let animationFrameId: number;

	const initScene = () => {
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

		renderer = new THREE.WebGLRenderer();
		renderer.setSize(window.innerWidth, window.innerHeight);
		document.body.appendChild(renderer.domElement);

		if (dev) {
			controls = new OrbitControls(camera, renderer.domElement);
		}

		// Textures
		const bricktexture = new THREE.TextureLoader().load('brick.png');
		bricktexture.wrapS = THREE.RepeatWrapping;
		bricktexture.wrapT = THREE.RepeatWrapping;
		bricktexture.repeat.set(5, 3);

		const darkconcretetexture = new THREE.TextureLoader().load('darkconcrete.png');
		darkconcretetexture.wrapS = THREE.RepeatWrapping;
		darkconcretetexture.wrapT = THREE.RepeatWrapping;
		darkconcretetexture.repeat.set(11, 5);

		const walnuttexture = new THREE.TextureLoader().load('walnut.png');
		walnuttexture.wrapS = THREE.RepeatWrapping;
		walnuttexture.wrapT = THREE.RepeatWrapping;
		walnuttexture.repeat.set(6, 3);

		const marbletexture = new THREE.TextureLoader().load('marble.png');
		marbletexture.wrapS = THREE.RepeatWrapping;
		marbletexture.wrapT = THREE.RepeatWrapping;
		marbletexture.repeat.set(2, 2);

		const darkwoodpaneltexture = new THREE.TextureLoader().load('panel.png');
		marbletexture.wrapS = THREE.RepeatWrapping;
		marbletexture.wrapT = THREE.RepeatWrapping;
		marbletexture.repeat.set(1, 1);

		// Elevator Doors
		const doorGeo = new THREE.PlaneGeometry(0.5, 2);

		doorRF = new Reflector(doorGeo, {
			textureWidth: (window.innerWidth * window.devicePixelRatio) / 16,
			textureHeight: (window.innerHeight * window.devicePixelRatio) / 16,
			color: new THREE.Color(0x7f7f7f)
		});
		doorRF.position.set(0.25, 1, -0.05);
		scene.add(doorRF);

		doorLF = new Reflector(doorGeo, {
			color: new THREE.Color(0x7f7f7f),
			textureWidth: (window.innerWidth * window.devicePixelRatio) / 16,
			textureHeight: (window.innerHeight * window.devicePixelRatio) / 16
		});
		doorLF.position.set(-0.25, 1, -0.05);
		scene.add(doorLF);

		doorRB = new Reflector(doorGeo, {
			color: new THREE.Color(0x7f7f7f),
			textureWidth: (window.innerWidth * window.devicePixelRatio) / 8,
			textureHeight: (window.innerHeight * window.devicePixelRatio) / 8
		});
		doorRB.position.set(0.75, 1, -0.1);
		doorRB.rotation.x = Math.PI;

		doorLB = new Reflector(doorGeo, {
			color: new THREE.Color(0x7f7f7f),
			textureWidth: (window.innerWidth * window.devicePixelRatio) / 4,
			textureHeight: (window.innerHeight * window.devicePixelRatio) / 4
		});
		doorLB.position.set(-0.75, 1, -0.1);
		doorLB.rotation.x = Math.PI;

		// Lobby Walls
		const wallGeo = new THREE.BoxGeometry(8, 2, 0.06);
		const wallMat = new THREE.MeshPhysicalMaterial({
			map: bricktexture,
			roughness: 0.5,
			metalness: 0.5,
			clearcoat: 0.4,
			clearcoatRoughness: 0.1
		});
		wallR = new THREE.Mesh(wallGeo, wallMat);
		wallR.position.set(4.5, 1, 0.02);
		scene.add(wallR);

		wallL = new THREE.Mesh(wallGeo, wallMat);
		wallL.position.set(-4.5, 1, 0);
		scene.add(wallL);

		const wallEGeo = new THREE.BoxGeometry(8, 2, 0.05);
		wallEnter = new THREE.Mesh(wallEGeo, wallMat);
		wallEnter.position.set(-8.5, 1, 4);
		wallEnter.rotation.y = Math.PI / 2;
		scene.add(wallEnter);

		wallExit = new THREE.Mesh(wallEGeo, wallMat);
		wallExit.position.set(8.5, 1, 4);
		wallExit.rotation.y = Math.PI / 2;
		scene.add(wallExit);

		const wallBGeo = new THREE.BoxGeometry(17, 2, 0.05);
		wallB = new THREE.Mesh(wallBGeo, wallMat);
		wallB.position.set(0, 1, 7.95);
		scene.add(wallB);

		const wallgeoT = new THREE.BoxGeometry(17, 4, 0.05);
		const wallmatT = new THREE.MeshPhysicalMaterial({
			map: darkconcretetexture,
			roughness: 0.5,
			metalness: 0.1,
			clearcoat: 0.1,
			clearcoatRoughness: 0.3
		});
		wallT = new THREE.Mesh(wallgeoT, wallmatT);
		wallT.position.set(0, 4, 0);
		scene.add(wallT);

		wallTB = new THREE.Mesh(wallgeoT, wallmatT);
		wallTB.position.set(0, 4, 7.95);
		scene.add(wallTB);

		const wallEgeoT = new THREE.BoxGeometry(8, 4, 0.05);
		wallEnterT = new THREE.Mesh(wallEgeoT, wallmatT);
		wallEnterT.position.set(-8.5, 4, 4);
		wallEnterT.rotation.y = Math.PI / 2;
		scene.add(wallEnterT);

		wallExitT = new THREE.Mesh(wallEgeoT, wallmatT);
		wallExitT.position.set(8.5, 4, 4);
		wallExitT.rotation.y = Math.PI / 2;
		scene.add(wallExitT);

		const lbFloorGeo = new THREE.PlaneGeometry(17, 8);
		const lbFloorMat = new THREE.MeshPhysicalMaterial({
			map: walnuttexture,
			roughness: 0.1,
			metalness: 0.4,
			clearcoat: 0.6,
			clearcoatRoughness: 0.1,
			reflectivity: 0.2
		});
		lbFloor = new THREE.Mesh(lbFloorGeo, lbFloorMat);
		lbFloor.position.set(0, 0, 3.95);
		lbFloor.rotation.x = -Math.PI / 2;
		scene.add(lbFloor);

		const lbCeilMat = new THREE.MeshPhysicalMaterial({
			color: 0xd9d9d9,
			roughness: 0.5,
			metalness: 0.1,
			clearcoat: 0.4,
			clearcoatRoughness: 0.1
		});
		lbCeil = new THREE.Mesh(lbFloorGeo, lbCeilMat);
		lbCeil.position.set(0, 6, 3.95);
		lbCeil.rotation.x = Math.PI / 2;
		scene.add(lbCeil);

		// Elevator Interior
		const elevWallGeo = new THREE.BoxGeometry(3, 3, 0.05);
		const elevwallMat = new THREE.MeshStandardMaterial({ color: 0xbbbbbb });
		elevWallR = new THREE.Mesh(elevWallGeo, elevwallMat);
		elevWallR.position.set(1.5, 1.5, -1.55);
		elevWallR.rotation.y = Math.PI / 2;
		scene.add(elevWallR);

		elevWallL = new THREE.Mesh(elevWallGeo, elevwallMat);
		elevWallL.position.set(-1.5, 1.5, -1.55);
		elevWallL.rotation.y = Math.PI / 2;
		scene.add(elevWallL);

		const elevWallgeoB = new THREE.BoxGeometry(3, 3, 0.05);
		elevWallB = new THREE.Mesh(elevWallgeoB, elevwallMat);
		elevWallB.position.set(0, 1.5, -3.05);
		scene.add(elevWallB);

		const elevWallgeoF = new THREE.BoxGeometry(1, 3, 0.05);
		elevWallFL = new THREE.Mesh(elevWallgeoF, elevwallMat);
		elevWallFL.position.set(1, 1.5, -0.1);
		scene.add(elevWallFL);

		elevWallFR = new THREE.Mesh(elevWallgeoF, elevwallMat);
		elevWallFR.position.set(-1, 1.5, -0.1);
		scene.add(elevWallFR);

		const elevWallgeoFT = new THREE.BoxGeometry(1, 1, 0.05);
		elevWallFT = new THREE.Mesh(elevWallgeoFT, elevwallMat);
		elevWallFT.position.set(0, 2.5, -0.1);
		scene.add(elevWallFT);

		const elevFloorGeo = new THREE.PlaneGeometry(3, 3);
		const elevFloorMat = new THREE.MeshPhysicalMaterial({
			map: marbletexture,
			roughness: 0.3,
			metalness: 0.5,
			clearcoat: 0.8,
			clearcoatRoughness: 0.4
		});
		elevFloor = new THREE.Mesh(elevFloorGeo, elevFloorMat);
		elevFloor.position.set(0, 0, -1.55);
		elevFloor.rotation.x = -Math.PI / 2;
		scene.add(elevFloor);

		const elevCeilMat = new THREE.MeshStandardMaterial({ color: 0xdddddd });
		elevCeil = new THREE.Mesh(elevFloorGeo, elevCeilMat);
		elevCeil.position.set(0, 3, -1.55);
		elevCeil.rotation.x = Math.PI / 2;
		scene.add(elevCeil);

		// Elevator Panels
		const elevPanelGeo = new THREE.BoxGeometry(1, 2.5, 0.07);
		const panelMat = new THREE.MeshPhysicalMaterial({
			map: darkwoodpaneltexture,
			roughness: 0.6,
			metalness: 0.3,
			clearcoat: 0.5,
			clearcoatRoughness: 0.3
		});
		elevPanel1 = new THREE.Mesh(elevPanelGeo, panelMat);
		elevPanel2 = new THREE.Mesh(elevPanelGeo, panelMat);
		elevPanel3 = new THREE.Mesh(elevPanelGeo, panelMat);
		elevPanel4 = new THREE.Mesh(elevPanelGeo, panelMat);
		elevPanel5 = new THREE.Mesh(elevPanelGeo, panelMat);
		elevPanel6 = new THREE.Mesh(elevPanelGeo, panelMat);

		elevPanel1.position.set(-1.42, 1.5, -1);
		elevPanel1.rotation.y = Math.PI / 2;
		scene.add(elevPanel1);

		elevPanel2.position.set(-1.42, 1.5, -2.2);
		elevPanel2.rotation.y = Math.PI / 2;
		scene.add(elevPanel2);

		elevPanel3.position.set(-0.75, 1.5, -3);
		scene.add(elevPanel3);

		elevPanel4.position.set(0.75, 1.5, -3);
		scene.add(elevPanel4);

		elevPanel5.position.set(1.42, 1.5, -2.2);
		elevPanel5.rotation.y = Math.PI / 2;
		scene.add(elevPanel5);

		elevPanel6.position.set(1.42, 1.5, -1);
		elevPanel6.rotation.y = Math.PI / 2;
		scene.add(elevPanel6);

		// Lighting
		const al = new THREE.AmbientLight(0xffffff, 0.25);
		scene.add(al);

		lpl1 = new THREE.PointLight(0xffffff, 18, 10, 1.2);
		lpl1.position.set(4, 6, 4);
		scene.add(lpl1);

		lpl2 = new THREE.PointLight(0xffffff, 18, 10, 1.2);
		lpl2.position.set(-4, 6, 4);
		scene.add(lpl2);

		lpl3 = new THREE.PointLight(0xffffff, 7, 5, 1.2);
		lpl3.position.set(0, 2.9, -1.5);
		scene.add(lpl3);

		camera.position.z = 3.5;
		camera.position.y = 1.3;

		// Temp next to wall
		// camera.position.set(1, 2, 1);

		// Temp inside elevator
		// camera.position.z = -2;
		// camera.rotation.y = Math.PI;

		// Temp bird view elevator
		// camera.position.z = -2;
		// camera.position.y = 6;
		// camera.rotation.x = -Math.PI / 2;

		// Temp bird view lobby
		// camera.position.z = 10;
		// camera.position.y = 16;
		// camera.rotation.x = -Math.PI / 2;

		if (dev) {
			controls.update();
		}
	};

	const renderScene = () => {
		renderer.render(scene, camera);
	};

	let doorsFRemoved = false;
	let lplRemoved = false;

	const animateScene = () => {
		animationFrameId = requestAnimationFrame(animateScene);

		const delta = clock.getDelta();
		const sec = clock.getElapsedTime();

		const initialWait = 5;

		if (!dev && sec > initialWait) {
			if (camera.position.z > -1.2) {
				camera.position.z -= 0.01;
			} else if (camera.rotation.y < Math.PI) {
				camera.rotation.y += 0.02;
				camera.position.x -= 0.005;
			}
		}

		if (sec < initialWait + 8 && sec > initialWait + 1 && doorLF.position.x > -0.75) {
			doorLF.position.x -= 0.005;
			doorRF.position.x += 0.005;
		} else if (!doorsFRemoved && sec >= initialWait + 8) {
			scene.remove(doorLF);
			scene.remove(doorRF);
			doorLF.geometry.dispose();
			doorRF.geometry.dispose();
			doorsFRemoved = true;
			scene.add(doorLB);
			scene.add(doorRB);
		} else if (sec > initialWait + 12 && doorLB.position.x < -0.25) {
			doorLB.position.x += 0.005;
			doorRB.position.x -= 0.005;
			if (!dev && !lplRemoved && doorLB.position.x >= -0.25) {
				scene.remove(lpl1);
				scene.remove(lpl2);

				// Remove Lobby
				scene.remove(lbCeil);
				lbCeil.geometry.dispose();
				scene.remove(lbFloor);
				lbFloor.geometry.dispose();
				scene.remove(wallEnterT);
				wallEnter.geometry.dispose();
				scene.remove(wallExitT);
				wallExitT.geometry.dispose();
				scene.remove(wallT);
				wallT.geometry.dispose();
				scene.remove(wallTB);
				wallTB.geometry.dispose();
				scene.remove(wallR);
				wallR.geometry.dispose();
				scene.remove(wallL);
				wallL.geometry.dispose();
				scene.remove(wallEnter);
				wallEnter.geometry.dispose();
				scene.remove(wallExit);
				wallExit.geometry.dispose();
				scene.remove(wallB);
				wallB.geometry.dispose();
			}
		}

		if (dev) {
			controls.update();
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
