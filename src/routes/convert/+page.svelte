<script lang="ts">
	import { onMount } from 'svelte';
	import { DateTime } from 'luxon';

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

	let allTimeZones: TimeZoneFeature[] = [];
	let sourceTimeZone: string = Intl.DateTimeFormat().resolvedOptions().timeZone;
	let selectedTargetTimeZones: string[] = [];
	let inputDateTime: string; // YYYY-MM-DDTHH:mm format

	let sourceSearchTerm: string = '';
	let filteredSourceTimeZones: TimeZoneFeature[] = [];

	function sanitizeString(str: string): string {
		return str.replace(/[^a-zA-Z0-9]/g, '');
	}

	$: {
		if (sourceSearchTerm) {
			const sanitizedSearchTerm = sanitizeString(sourceSearchTerm).toLowerCase();
			filteredSourceTimeZones = allTimeZones.filter((tz) =>
				sanitizeString(tz.properties.tzid).toLowerCase().includes(sanitizedSearchTerm)
			);
		} else {
			filteredSourceTimeZones = [];
		}
	}

	function selectSourceTimeZone(tzid: string) {
		sourceTimeZone = tzid;
		sourceSearchTerm = ''; // Clear search after selection
	}

	let searchTerm: string = '';
	let filteredTimeZones: TimeZoneFeature[] = [];

	$: {
		if (searchTerm) {
			const sanitizedSearchTerm = sanitizeString(searchTerm).toLowerCase();
			filteredTimeZones = allTimeZones.filter((tz) =>
				sanitizeString(tz.properties.tzid).toLowerCase().includes(sanitizedSearchTerm)
			);
		} else {
			filteredTimeZones = [];
		}
	}

	function addTimeZone(tzid: string) {
		if (!selectedTargetTimeZones.includes(tzid)) {
			selectedTargetTimeZones = [...selectedTargetTimeZones, tzid];
			searchTerm = ''; // Clear search after adding
		}
	}

	function removeTimeZone(tzid: string) {
		selectedTargetTimeZones = selectedTargetTimeZones.filter((t) => t !== tzid);
	}

	let convertedTimes: { tzid: string; dateTime: string }[] = [];

	onMount(async () => {
		const response = await fetch('/data/2025b-combined-simplified.json');
		const data = await response.json();
		allTimeZones = data.features.sort((a: TimeZoneFeature, b: TimeZoneFeature) =>
			a.properties.tzid.localeCompare(b.properties.tzid)
		);

		// Set default input date and time to now in user's local timezone
		inputDateTime = DateTime.now().toFormat("yyyy-MM-dd'T'HH:mm");
	});

	function convertTime() {
		if (!inputDateTime) {
			alert('Please enter a valid date and time.');
			return;
		}

		const sourceDateTime = DateTime.fromISO(inputDateTime, { zone: sourceTimeZone });

		if (!sourceDateTime.isValid) {
			alert('Invalid date/time or time zone. Please check your input.');
			return;
		}

		convertedTimes = selectedTargetTimeZones.map((tzid) => {
			try {
				const targetDateTime = sourceDateTime.setZone(tzid);
				if (targetDateTime.isValid) {
					return {
						tzid: tzid,
						dateTime: targetDateTime.toLocaleString(DateTime.DATETIME_MED_WITH_WEEKDAY)
					};
				} else {
					throw new Error('Invalid target time zone');
				}
			} catch (e) {
				console.error(`Error converting time for ${tzid}:`, e);
				return {
					tzid: tzid,
					dateTime: 'N/A'
				};
			}
		});
	}
</script>

<svelte:head>
	<title>Convert Date & Time | zones.arilyn.cc</title>
</svelte:head>

<div class="container mt-4">
	<h1>Convert Date & Time</h1>
	<p class="text-muted mb-3">Convert a date and time between time zones.</p>

	<div class="form-elements-wrapper">
		<div class="mb-3">
			<label for="inputDateTime" class="form-label">Source Date and Time:</label>
			<input
				type="datetime-local"
				class="form-control"
				id="inputDateTime"
				bind:value={inputDateTime}
			/>
		</div>

		<div class="mb-3">
			<label for="sourceTimeZoneSearch" class="form-label">Source Time Zone:</label>
			<input
				type="text"
				class="form-control mb-2"
				id="sourceTimeZoneSearch"
				placeholder="Search for a time zone..."
				bind:value={sourceSearchTerm}
			/>
			{#if sourceSearchTerm && filteredSourceTimeZones.length > 0}
				<ul class="list-group mb-2" style="max-height: 150px; overflow-y: auto;">
					{#each filteredSourceTimeZones as tz (tz.properties.tzid)}
						<li class="list-group-item p-0">
							<button
								type="button"
								class="list-group-item list-group-item-action text-start w-100"
								on:click={() => selectSourceTimeZone(tz.properties.tzid)}
							>
								{tz.properties.tzid}
							</button>
						</li>
					{/each}
				</ul>
			{:else if sourceSearchTerm && filteredSourceTimeZones.length === 0}
				<div class="alert alert-warning">No time zones found matching search.</div>
			{/if}
			<div class="alert alert-secondary">
				Currently selected: <strong>{sourceTimeZone}</strong>
			</div>
		</div>

		<div class="mb-3">
			<label for="targetTimeZoneSearch" class="form-label">Target Time Zones:</label>
			<input
				type="text"
				class="form-control mb-2"
				id="targetTimeZoneSearch"
				placeholder="Search for a time zone..."
				bind:value={searchTerm}
			/>
			{#if searchTerm && filteredTimeZones.length > 0}
				<ul class="list-group mb-2" style="max-height: 150px; overflow-y: auto;">
					{#each filteredTimeZones as tz (tz.properties.tzid)}
						<li class="list-group-item p-0">
							<button
								type="button"
								class="list-group-item list-group-item-action text-start w-100"
								on:click={() => addTimeZone(tz.properties.tzid)}
							>
								{tz.properties.tzid}
							</button>
						</li>
					{/each}
				</ul>
			{:else if searchTerm && filteredTimeZones.length === 0}
				<div class="alert alert-warning">No time zones found matching search.</div>
			{/if}

			{#if selectedTargetTimeZones.length > 0}
				<ul class="list-group">
					{#each selectedTargetTimeZones as tzid (tzid)}
						<li class="list-group-item d-flex justify-content-between align-items-center">
							{tzid}
							<button
								type="button"
								class="btn btn-sm btn-danger"
								on:click={() => removeTimeZone(tzid)}>Remove</button
							>
						</li>
					{/each}
				</ul>
			{:else}
				<div class="alert alert-info">No target time zones selected.</div>
			{/if}
		</div>

		<div class="mb-3 text-center">
			<button class="btn btn-primary mb-4" on:click={convertTime}>Convert</button>
		</div>
	</div>

	{#if convertedTimes.length > 0}
		<h2 class="mt-4">Converted Times:</h2>
		<div class="table-responsive">
			<table class="table table-striped table-hover">
				<thead class="table-dark">
					<tr>
						<th scope="col">Time Zone</th>
						<th scope="col">Converted Date/Time</th>
					</tr>
				</thead>
				<tbody>
					{#each convertedTimes as item (item.tzid)}
						<tr>
							<td>{item.tzid}</td>
							<td>{item.dateTime}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	{/if}
</div>

<style>
	.table-responsive table th,
	.table-responsive table td {
		white-space: nowrap;
	}

	@media (min-width: 992px) {
		.form-elements-wrapper {
			max-width: 500px; /* Adjust as needed */
			margin: 0 auto;
		}
	}
</style>
