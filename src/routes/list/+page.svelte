<script lang="ts">
	import { onMount } from 'svelte';
	import { DateTime } from 'luxon';

	let now = DateTime.now();

	interface TimeZoneFeature {
		type: 'Feature';
		geometry: {
			type: 'Polygon' | 'MultiPolygon';
			coordinates: number[][][] | number[][][][];
		};
		properties: {
			tzid: string;
		};
	}

	let timeZones: TimeZoneFeature[] = [];
	let groupedTimeZones: { [key: string]: TimeZoneFeature[] } = {};
	let regions: string[] = [];
	let selectedRegion = 'All';

	onMount(async () => {
		const response = await fetch('/data/2025b-combined-simplified.json');
		const data = await response.json();
		timeZones = data.features.sort((a: TimeZoneFeature, b: TimeZoneFeature) =>
			a.properties.tzid.localeCompare(b.properties.tzid)
		);

		groupTimeZones();
	});

	function groupTimeZones() {
		const groups: { [key: string]: TimeZoneFeature[] } = {};
		timeZones.forEach((tz) => {
			const region = tz.properties.tzid.split('/')[0];
			if (!groups[region]) {
				groups[region] = [];
			}
			groups[region].push(tz);
		});
		groupedTimeZones = groups;
		regions = Object.keys(groups).sort();
	}

	function selectRegion(region: string) {
		selectedRegion = region;
	}

	$: filteredTimeZones = (() => {
		if (selectedRegion === 'All') {
			return timeZones;
		}
		return groupedTimeZones[selectedRegion] || [];
	})();

	function getZoneInfo(tzid: string) {
		try {
			const zonedNow = now.setZone(tzid);
			return {
				time: zonedNow.toLocaleString(DateTime.DATETIME_MED_WITH_WEEKDAY),
				offset: `UTC${zonedNow.toFormat('Z')}`,
				dst: zonedNow.isInDST ? 'Yes' : 'No'
			};
		} catch {
			return {
				time: 'N/A',
				offset: 'N/A',
				dst: 'N/A'
			};
		}
	}
</script>

<div class="container mt-4">
	<h1>Time Zone List</h1>
	<p class="text-muted mb-3">
		List of time zones. Choose a region below to filter. Note: DST = Daylight Savings Time
	</p>

	<div class="mb-4">
		<button
			class="btn me-2 mb-2 {selectedRegion === 'All' ? 'btn-primary' : 'btn-light'}"
			on:click={() => selectRegion('All')}>All</button
		>
		{#each regions as region (region)}
			<button
				class="btn me-2 mb-2 {selectedRegion === region ? 'btn-primary' : 'btn-light'}"
				on:click={() => selectRegion(region)}>{region}</button
			>
		{/each}
	</div>

	<div class="table-responsive">
		<table class="table table-striped table-hover">
			<thead class="table-dark">
				<tr>
					<th scope="col">Time Zone</th>
					<th scope="col">Time</th>
					<th scope="col">UTC Offset</th>
					<th scope="col">In DST</th>
				</tr>
			</thead>
			<tbody>
				{#each filteredTimeZones as tz (tz.properties.tzid)}
					{@const zoneInfo = getZoneInfo(tz.properties.tzid)}
					<tr>
						<td>{tz.properties.tzid}</td>
						<td>{zoneInfo.time}</td>
						<td>{zoneInfo.offset}</td>
						<td>{zoneInfo.dst}</td>
					</tr>
				{/each}
			</tbody>
		</table>
	</div>
</div>

<style>
	.table-responsive table th,
	.table-responsive table td {
		white-space: nowrap;
	}
</style>
