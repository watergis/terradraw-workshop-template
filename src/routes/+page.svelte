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
		TerraDrawPolygonMode,
		TerraDrawSelectMode,
		TerraDrawExtend
	} from 'terra-draw';
	import { TerraDrawMapLibreGLAdapter } from 'terra-draw-maplibre-gl-adapter';

	let mapContainer: HTMLDivElement | undefined = $state();
	let map: Map | undefined = $state();
	let draw: TerraDraw | undefined = $state();

	let selectedFeature: string = $state('');

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

			// Add the Point, LineString, Polygon and Select Mode
			modes: [
				new TerraDrawPointMode({
					styles: {
						pointColor: '#FFFFFF',
						pointWidth: 5,
						pointOutlineColor: '#FF0000',
						pointOutlineWidth: 1
					}
				}),
				new TerraDrawLineStringMode({
					styles: {
						lineStringColor: '#FF0000',
						lineStringWidth: 2,
						closingPointColor: '#FFFFFF',
						closingPointWidth: 3,
						closingPointOutlineColor: '#FF0000',
						closingPointOutlineWidth: 1
					}
				}),
				new TerraDrawPolygonMode({
					styles: {
						fillColor: '#FFFFFF',
						fillOpacity: 0.7,
						outlineColor: '#FF0000',
						outlineWidth: 2,
						closingPointColor: '#FFFFFF',
						closingPointWidth: 3,
						closingPointOutlineColor: '#FF0000',
						closingPointOutlineWidth: 1
					}
				}),
				// Add Select mode with feature and coordinate editing enabled
				new TerraDrawSelectMode({
					styles: {
						// Point colour
						selectedPointColor: '#FF0000',
						selectedPointWidth: 7,
						selectedPointOutlineColor: '#FFFF00',
						selectedPointOutlineWidth: 2,
						// LineString colour
						selectedLineStringColor: '#FFFF00',
						selectedLineStringWidth: 4,
						// Polygon colour
						selectedPolygonColor: '#FF0000',
						selectedPolygonFillOpacity: 0.7,
						selectedPolygonOutlineColor: '#FFFF00',
						selectedPolygonOutlineWidth: 4,
						// Selection point colour
						selectionPointColor: '#FF0000',
						selectionPointWidth: 8,
						selectionPointOutlineColor: '#FFFF00',
						selectionPointOutlineWidth: 2,
						// Midpoint colour
						midPointColor: '#FF0000',
						midPointWidth: 6,
						midPointOutlineColor: '#FFFF00',
						midPointOutlineWidth: 2
					},
					flags: {
						point: {
							feature: {
								draggable: true
							}
						},
						linestring: {
							feature: {
								draggable: true,
								rotateable: true,
								coordinates: {
									midpoints: true,
									draggable: true,
									deletable: true
								}
							}
						},
						polygon: {
							feature: {
								draggable: true,
								rotateable: true,
								coordinates: {
									midpoints: true,
									draggable: true,
									deletable: true
								}
							}
						}
					}
				})
			]
		});

		map.once('load', () => {
			// Start drawing
			draw?.start();

			draw?.on('change', (ids, type) => {
				if (!draw) return;
				if (type === 'create') {
					// After creating a feature
					for (const id of ids) {
						// Do something
						console.log(`change: Feature created: ${id}`);
					}
				} else if (type === 'update') {
					//After updating a feature
					for (const id of ids) {
						console.log(`change: Feature updated: ${id}`);
					}
				} else if (type === 'delete') {
					//After deleting a feature
					for (const id of ids) {
						console.log(`change: Feature deleted: ${id}`);
					}
				} else if (type === 'styling') {
					//After styling is changed
					for (const id of ids) {
						console.log(`change: Feature styling updated: ${id}`);
					}
				}
			});

			draw?.on(
				'finish',
				(id: TerraDrawExtend.FeatureId, context: { action: string; mode: string }) => {
					if (context.action === 'draw') {
						// Do something for draw finish event for a feature
						console.log(`finish: Feature drawn: ${id} in mode: ${context.mode}`);
					} else if (context.action === 'dragFeature') {
						// Do something for a drag finish event for a feature
						console.log(`finish: Feature dragged: ${id} in mode: ${context.mode}`);
					} else if (context.action === 'dragCoordinate') {
						// Do something for a drag finish event for a coordinate
						console.log(`finish: Coordinate dragged in feature: ${id} in mode: ${context.mode}`);
					} else if (context.action === 'dragCoordinateResize') {
						// Do something for a drag finish event for resizing a feature
						console.log(`finish: Coordinate resized in feature: ${id} in mode: ${context.mode}`);
					}
				}
			);

			draw?.on('select', (id: TerraDrawExtend.FeatureId) => {
				console.log(`selected: ${id}`);
				const feature = draw?.getSnapshotFeature(id);
				selectedFeature = feature ? JSON.stringify(feature, null, 2) : '';
			});

			draw?.on('deselect', () => {
				console.log(`deselected`);
				selectedFeature = '';
			});
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

		<!-- Add select mode button here -->
		<hr />
		<button onclick={() => handleModeClick('select')}>Select</button>

		<!-- Add text area for selected feature here -->
		<hr />
		<label for="selected-feature-geojson">Selected Feature GeoJSON:</label>
		<textarea
			id="selected-feature-geojson"
			bind:value={selectedFeature}
			style="width: 100%; resize: vertical;"
			rows="10"
			readonly
		></textarea>
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
