<script lang="ts">
	import {
		AttributionControl,
		GeolocateControl,
		GlobeControl,
		Map,
		NavigationControl,
		ScaleControl
	} from 'maplibre-gl';
	import { onMount } from 'svelte';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import {
		TerraDraw,
		TerraDrawPointMode,
		TerraDrawLineStringMode,
		TerraDrawPolygonMode
	} from 'terra-draw';
	import { TerraDrawMapLibreGLAdapter } from 'terra-draw-maplibre-gl-adapter';

	let mapContainer: HTMLDivElement | undefined = $state();
	let map: Map | undefined = $state();
	let draw: TerraDraw | undefined = $state();

	onMount(() => {
		if (!mapContainer) return;
		map = new Map({
			container: mapContainer,
			style: {
				version: 8,
				name: 'OpnenStreetMap Raster',
				sources: {
					osm: {
						type: 'raster',
						tiles: ['https://tile.openstreetmap.jp/{z}/{x}/{y}.png'],
						tileSize: 256,
						maxzoom: 19,
						attribution:
							'<a href="https://www.openstreetmap.org/copyright" target="_blank">&copy; OpenStreetMap contributors</a>'
					}
				},
				layers: [
					{
						id: 'osm',
						type: 'raster',
						source: 'osm'
					}
				]
			},
			center: [174.7629, -36.8587],
			zoom: 11,
			hash: true,
			attributionControl: false
		});
		map.addControl(new NavigationControl(), 'top-right');
		map.addControl(new GlobeControl(), 'top-right');
		map.addControl(
			new GeolocateControl({
				positionOptions: { enableHighAccuracy: true },
				trackUserLocation: true
			}),
			'top-right'
		);
		map.addControl(new ScaleControl({ maxWidth: 80, unit: 'metric' }), 'bottom-left');
		map.addControl(new AttributionControl({ compact: true }), 'bottom-right');

		draw = new TerraDraw({
			// Using the MapLibre Adapter
			adapter: new TerraDrawMapLibreGLAdapter({ map }),

			// Add the Point, LineSdtring and Polygon Mode
			modes: [new TerraDrawPointMode(), new TerraDrawLineStringMode(), new TerraDrawPolygonMode()]
		});

		map.once('load', () => {
			// Start drawing
			draw?.start();
		});
	});

	const handleModeClick = (mode: string) => {
		draw?.setMode(mode);
	};

	const handleClearClick = () => {
		draw?.clear();
	};
</script>

<div class="main">
	<aside class="sidebar">
		<!-- Use this space for adding additional elements for workshop -->
		<button onclick={() => handleModeClick('point')}>Point</button>
		<button onclick={() => handleModeClick('linestring')}>Line</button>
		<button onclick={() => handleModeClick('polygon')}>Polygon</button>
		<button onclick={handleClearClick}>Clear</button>
	</aside>
	<div class="map" bind:this={mapContainer}></div>
</div>

<style lang="scss">
	.main {
		display: flex;
		height: 100vh;
		width: 100vw;

		.sidebar {
			width: 260px;
			background: #f4f4f4;
			border-right: 1px solid #ddd;
			padding: 1rem;
			box-sizing: border-box;
			overflow-y: auto;
		}
		.map {
			flex: 1;
			height: 100%;
			width: 100%;
		}
	}
</style>
