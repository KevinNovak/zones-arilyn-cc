<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import { page } from '$app/state';
	import { goto } from '$app/navigation';
	import type { Map, LatLngExpression, Path, GeoJSON } from 'leaflet';
	import { DateTime } from 'luxon';

	// Constants
	const MAP_DEFAULT_LATLONG: LatLngExpression = [25, 10];
	const MAP_DEFAULT_ZOOM = 2;
	const MAP_MIN_ZOOM = 1;
	const MAP_MAX_ZOOM = 15;
	const GEOJSON_PATH = 'data/2025b-combined-simplified.json';
	const TILES_URL = 'https://tile.openstreetmap.org/{z}/{x}/{y}.png';
	const TILES_COPYRIGHT_URL = 'http://www.openstreetmap.org/copyright';
	const TILES_ATTRIBUTION = `&copy; <a href="${TILES_COPYRIGHT_URL}">OpenStreetMap</a>`;

	// Style definitions
	const ZONE_STYLE_DEFAULT = {
		color: '#ffffff',
		weight: 2,
		opacity: 1,
		dashArray: '3',
		fillColor: '#fd8d3c',
		fillOpacity: 0.1
	};
	const ZONE_STYLE_HOVER = {
		color: '#666666',
		weight: 5,
		opacity: 1,
		dashArray: '',
		fillColor: '#fd8d3c',
		fillOpacity: 0.3
	};
	const ZONE_STYLE_SELECTED = {
		color: '#666666',
		weight: 5,
		opacity: 1,
		dashArray: '',
		fillColor: '#0000ff',
		fillOpacity: 0.3
	};

	let mapContainer: HTMLDivElement;
	let map: Map;
	let selectedTimeZone: string = '';
	let clickedPath: Path | null = null;
	let copySuccess: boolean = false;
	let geoJsonLayer: GeoJSON | null = null;

	onMount(async () => {
		// Only run on the client side
		if (!browser) return;

		// Dynamic import of Leaflet to avoid SSR issues
		const L = (await import('leaflet')).default;

		// Initialize the map
		map = L.map(mapContainer).setView(MAP_DEFAULT_LATLONG, MAP_DEFAULT_ZOOM);

		// Create map tile layer
		L.tileLayer(TILES_URL, {
			minZoom: MAP_MIN_ZOOM,
			maxZoom: MAP_MAX_ZOOM,
			attribution: TILES_ATTRIBUTION
		}).addTo(map);

		// Load and add GeoJSON layer
		const geoJsonData = await fetch(GEOJSON_PATH);
		const geoJson = await geoJsonData.json();

		geoJsonLayer = L.geoJSON(geoJson, {
			style: ZONE_STYLE_DEFAULT,
			onEachFeature: (feature, layer) => {
				const path = layer as Path;
				if (feature.properties && feature.properties.tzid) {
					// Add tooltip
					path.bindTooltip(feature.properties.tzid, { sticky: true });

					// Add hover events
					path.on('mouseover', () => {
						const timeZone = feature.properties.tzid;

						// Set tooltip content
						const time = DateTime.now()
							.setZone(timeZone)
							.toLocaleString(DateTime.DATETIME_MED_WITH_WEEKDAY);
						const tooltipContent = `<strong>${timeZone}</strong><br><small>${time}</small>`;
						path.setTooltipContent(tooltipContent);

						// Change style of the hovered path
						if (path !== clickedPath) {
							path.setStyle(ZONE_STYLE_HOVER);
							path.bringToFront();
						}
					});

					path.on('mouseout', () => {
						// Change style of the hovered path back to default
						if (path !== clickedPath) {
							path.setStyle(ZONE_STYLE_DEFAULT);
							path.bringToBack();
						}
					});

					// Add click event to update the selected time zone and style
					path.on('click', () => {
						// Reset previously clicked path style
						if (clickedPath && clickedPath !== path) {
							clickedPath.setStyle(ZONE_STYLE_DEFAULT);
							clickedPath.bringToBack();
						}

						// Change style of the clicked path
						path.setStyle(ZONE_STYLE_SELECTED);
						path.bringToFront();

						// Update clicked layer reference
						clickedPath = path;

						// Update selected time zone
						selectedTimeZone = feature.properties.tzid;

						// Fit map to bounds of the selected time zone
						map.fitBounds(path.getBounds(), { padding: [10, 10] });

						// Update URL
						goto(`?tz=${selectedTimeZone}`, { replaceState: true, noScroll: true });
					});
				}
			}
		}).addTo(map);

		// Check for time zone in URL
		const urlTimeZone = page.url.searchParams.get('tz');
		if (urlTimeZone) {
			const layer = findLayerByTimeZone(urlTimeZone);
			if (layer) {
				layer.fire('click');
			}
		}
	});

	// Find layer by time zone
	function findLayerByTimeZone(timeZone: string): Path | undefined {
		if (!geoJsonLayer) return undefined;

		return geoJsonLayer.getLayers().find((layer) => {
			const path = layer as Path;
			return path.feature?.properties?.tzid === timeZone;
		}) as Path | undefined;
	}

	// Detect and select user's time zone
	function detectTimeZone() {
		const userTimeZone = Intl.DateTimeFormat().resolvedOptions().timeZone;
		const layer = findLayerByTimeZone(userTimeZone);

		if (!layer) {
			alert(`Could not find your time zone on the map: ${userTimeZone}`);
			return;
		}

		layer.fire('click');
	}

	// Copy time zone to clipboard
	async function copyTimeZoneToClipboard() {
		if (!selectedTimeZone) {
			return;
		}

		if (!navigator.clipboard) {
			alert('Clipboard not available.');
			console.error('Clipboard API not available.');
			return;
		}

		try {
			await navigator.clipboard.writeText(selectedTimeZone);
			copySuccess = true;
			setTimeout(() => {
				copySuccess = false;
			}, 2000);
		} catch (err) {
			alert('Failed to copy to clipboard.');
			console.error('Failed to copy to clipboard:', err);
		}
	}
</script>

<svelte:head>
	<title>Time Zone Map | zones.arilyn.cc</title>
	<link
		rel="stylesheet"
		href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
		integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
		crossorigin=""
	/>
</svelte:head>

<h1>Time Zone Map</h1>
<p class="text-muted mb-3">
	Explore time zones on the map below. Click on a region to select it's time zone, or use the "Auto"
	button to find your time zone automatically.
</p>

<!-- Time Zone Display -->
<div class="mb-3 p-2 border rounded">
	<div class="d-flex justify-content-between align-items-center">
		<div class="d-flex align-items-center gap-2">
			<button
				class="btn btn-sm btn-outline-secondary"
				on:click={detectTimeZone}
				title="Detect your time zone"
			>
				<i class="bi bi-crosshair"></i> Auto
			</button>
			<strong>Time Zone:</strong>
			<span>{selectedTimeZone || 'None selected'}</span>
			{#if selectedTimeZone}
				<button
					class="btn btn-sm btn-outline-secondary"
					on:click={copyTimeZoneToClipboard}
					title="Copy time zone to clipboard"
				>
					{#if copySuccess}
						<span class="text-success"><i class="bi bi-check-lg"></i></span>
					{:else}
						<i class="bi bi-files"></i> Copy
					{/if}
				</button>
			{/if}
		</div>
	</div>
</div>

<div
	bind:this={mapContainer}
	id="map"
	style="height: 500px; width: 100%; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1);"
></div>

<style>
	:global(.leaflet-container) {
		z-index: 1;
	}
</style>
