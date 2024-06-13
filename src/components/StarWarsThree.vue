<template>
	<div
		ref="threeContainer"
		id="threeContainer"
		@mousemove="onMouseMove"
	></div>
</template>

<script>
	import * as THREE from 'three';
	import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js';
	import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js';
	import TWEEN from '@tweenjs/tween.js';

	export default {
		name: 'StarWarsThree',
		props: {
			characters: {
				type: Array,
				default: () => [],
			},
		},
		watch: {
			characters: {
				handler(newData) {
					if (newData.length === 2) {
						this.$nextTick(() => {
							this.createScene(newData);
						});
					}
				},
				immediate: true,
				deep: true,
			},
		},
		data() {
			return {
				mouseX: 0,
				mouseY: 0,
			};
		},
		methods: {
			onMouseMove(event) {
				const rect = this.$refs.threeContainer.getBoundingClientRect();
				this.mouseX = ((event.clientX - rect.left) / rect.width) * 2 - 1;
				this.mouseY = -((event.clientY - rect.top) / rect.height) * 2 + 1;
			},
			createScene(data) {
				// Clear previous scene
				const container = this.$refs.threeContainer;
				while (container.firstChild) {
					container.removeChild(container.firstChild);
				}

				const width = container.clientWidth || window.innerWidth;
				const height = container.clientHeight || 400;

				const scene = new THREE.Scene();
				const camera = new THREE.PerspectiveCamera(
					75,
					width / height,
					0.1,
					1000
				);
				const renderer = new THREE.WebGLRenderer({ antialias: true });
				renderer.setPixelRatio(window.devicePixelRatio);
				renderer.setSize(width, height);
				container.appendChild(renderer.domElement);

				const colors = ['#43a5cf', '#e42f43'];
				const barHeight = 6;
				const barDepth = 2;
				const barSpacing = 20;
				const leftMargin = -50; // Adjusted for better alignment
				const labelOffset = -50; // Smaller left offset for labels
				const maxBarWidth = 140; // Adjusted for better fit

				const fontLoader = new FontLoader();
				fontLoader.load(
					'https://threejs.org/examples/fonts/helvetiker_regular.typeface.json',
					font => {
						// Calculate the maximum character height
						const maxHeight = Math.max(
							...data.map(character => character.height)
						);

						data.forEach((character, index) => {
							const barWidth = (character.height / maxHeight) * maxBarWidth;

							const geometry = new THREE.BoxGeometry(
								barWidth,
								barHeight,
								barDepth
							);
							const material = new THREE.MeshBasicMaterial({
								color: colors[index],
							});
							const bar = new THREE.Mesh(geometry, material);
							bar.position.x = barWidth / 2 + leftMargin;
							bar.position.y =
								-index * barSpacing + (data.length - 1) * (barSpacing / 2);
							scene.add(bar);

							const textGeometry = new TextGeometry(character.name, {
								font: font,
								size: 4, // Size of the text
								depth: 0.1, // Depth of the text extrusion
								curveSegments: 12, // Number of points on the curves
								bevelEnabled: false, // Whether or not to enable beveling
								wrapWidth: 30, // Maximum width for wrapping
							});
							const textMaterial = new THREE.MeshBasicMaterial({
								color: '#ffe81f',
							});
							const textMesh = new THREE.Mesh(textGeometry, textMaterial);
							textMesh.position.x = leftMargin + labelOffset; // Adjusted to fit better
							textMesh.position.y = bar.position.y - barHeight / 2 + 2;
							scene.add(textMesh);
						});

						const heightLabels = [0, 50, 100, 150, 200];
						heightLabels.forEach(height => {
							const labelGeometry = new TextGeometry(
								this.formatHeight(height),
								{
									font: font,
									size: 3, // Size of the text
									depth: 0.1, // Depth of the text extrusion
									curveSegments: 12, // Number of points on the curves
									bevelEnabled: false, // Whether or not to enable beveling
									wrapWidth: 15, // Maximum width for wrapping
								}
							);
							const labelMaterial = new THREE.MeshBasicMaterial({
								color: '#ffe81f',
							});
							const labelMesh = new THREE.Mesh(labelGeometry, labelMaterial);
							labelMesh.position.x =
								(height / maxHeight) * maxBarWidth + leftMargin; // Adjusted for better fit
							labelMesh.position.y =
								-(data.length - 1) * (barSpacing / 2) - barHeight - 10;
							scene.add(labelMesh);

							const tickGeometry = new THREE.BoxGeometry(0.5, 0.5, 1);
							const tickMaterial = new THREE.MeshBasicMaterial({
								color: '#ffe81f',
							});
							const tickMesh = new THREE.Mesh(tickGeometry, tickMaterial);
							tickMesh.position.x =
								(height / maxHeight) * maxBarWidth + leftMargin; // Adjusted for better fit
							tickMesh.position.y =
								-(data.length - 1) * (barSpacing / 2) - barHeight / 2 - 10;
							scene.add(tickMesh);
						});

						camera.position.set(0, 0, 80);
						camera.lookAt(new THREE.Vector3(0, 0, 0));

						const animate = () => {
							requestAnimationFrame(animate);
							TWEEN.update();

							// Update scene rotation based on mouse position
							scene.rotation.y = this.mouseX * 0.03;
							scene.rotation.x = this.mouseY * 0.03;

							renderer.render(scene, camera);
						};

						animate();
					},
					undefined,
					function (error) {
						console.error('Error loading font:', error);
					}
				);
			},
			formatHeight(cm) {
				const totalInches = cm * 0.393701;
				const feet = Math.floor(totalInches / 12);
				const inches = Math.round(totalInches % 12);
				return `${feet}' ${inches}"`;
			},
		},
	};
</script>

<style>
	#threeContainer {
		width: 100%;
		height: 400px;
		border: 2px solid yellow;
	}
</style>
