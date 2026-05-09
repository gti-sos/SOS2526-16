<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    async function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) return resolve();
            const script = document.createElement('script');
            script.src = src; script.onload = resolve; document.head.appendChild(script);
        });
    }

    onMount(async () => {
        if (!browser) return;
        try {
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/funnel.js"); 
            const Highcharts = window.Highcharts;

            const res = await fetch('https://rickandmortyapi.com/api/character');
            const data = await res.json();
            
            
            const counts = data.results.reduce((acc, char) => {
                acc[char.species] = (acc[char.species] || 0) + 1;
                return acc;
            }, {});

            const chartData = Object.entries(counts).sort((a, b) => b[1] - a[1]);

            Highcharts.chart('container-funnel', {
                chart: { type: 'funnel' },
                title: { text: 'Distribución de Especies (Rick & Morty)' },
                plotOptions: {
                    funnel: {
                        dataLabels: { enabled: true, format: '<b>{point.name}</b>: {point.y}' },
                        neckWidth: '30%', neckHeight: '25%', width: '70%'
                    }
                },
                series: [{ name: 'Personajes', data: chartData }]
            });
        } catch (e) { console.error(e); }
    });
</script>

<div style="text-align: center; padding: 20px;">
    <h1>Integración Externa 5: Rick & Morty (Funnel Chart)</h1>
    <div id="container-funnel" style="height: 500px;"></div>
</div>