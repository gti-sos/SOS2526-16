<script>
	import { onMount } from 'svelte';

	let loading = true;
	let error = '';

	const PROXY_URL =
		'/api/v1/global-ev-charging-infrastructures/proxy/brands';

	onMount(async () => {
		try {
			const res = await fetch(PROXY_URL);

			if (!res.ok) {
				throw new Error(`Error HTTP: ${res.status}`);
			}

			const json = await res.json();
			console.log('Brands API response:', json);

			const brands = Array.isArray(json) ? json : json.data || json.result || [];

			createChart(brands);
		} catch (e) {
			error = e.message;
		} finally {
			loading = false;
		}
	});

	function parseBrandValue(value) {
		return Number(String(value).replace('$', '').replace(',', '')) || 0;
	}

	function createChart(brands) {
		const Highcharts = window.Highcharts;

		const treemapData = brands
			.slice(0, 30)
			.map((brand) => ({
				name: brand.Brand,
				value: parseBrandValue(brand['Brand Value ($B)']),
				colorValue: Number(brand.Rank),
				custom: {
					rank: brand.Rank,
					country: brand.Country,
					sector: brand.Sector
				}
			}))
			.filter((b) => b.value > 0);

		Highcharts.chart('container', {
			chart: {
				type: 'treemap'
			},

			colorAxis: {
				minColor: '#FFFFFF',
				maxColor: Highcharts.getOptions().colors[0]
			},

			title: {
				text: 'Top Brand Values Treemap'
			},

			subtitle: {
				text: 'Brand valuation data from RapidAPI through a custom proxy'
			},

			tooltip: {
				useHTML: true,
				pointFormat: `
					<b>{point.name}</b><br/>
					Brand value: {point.value} $B<br/>
					Rank: {point.custom.rank}<br/>
					Country: {point.custom.country}<br/>
					Sector: {point.custom.sector}
				`
			},

			series: [
				{
					type: 'treemap',
					layoutAlgorithm: 'squarified',
					clip: false,
					dataLabels: {
						enabled: true,
						format: '{point.name}',
						style: {
							textOutline: 'none'
						}
					},
					data: treemapData
				}
			]
		});
	}
</script>

<svelte:head>
	<script src="https://code.highcharts.com/highcharts.js"></script>
	<script src="https://code.highcharts.com/modules/treemap.js"></script>
</svelte:head>

<main>
	<h1>Top Brand Names Valuation</h1>

	<p>
		This integration retrieves brand valuation data from RapidAPI using a custom
		proxy and displays it with a Highcharts treemap.
	</p>

	{#if loading}
		<p>Loading brand data...</p>
	{/if}

	{#if error}
		<p class="error">Error: {error}</p>
	{/if}

	<div id="container"></div>
</main>
