<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let isLoading = true;

    async function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) return resolve();
            const script = document.createElement('script');
            script.src = src;
            script.onload = resolve;
            script.onerror = reject;
            document.head.appendChild(script);
        });
    }

    onMount(async () => {
        if (!browser) return;

        try {
       
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/funnel.js"); // 👈 NECESARIO PARA PIRÁMIDE
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            const Highcharts = window.Highcharts;

            
            const res = await fetch('/api/v1/proxy/spacex');
            const data = await res.json();
            
            
            const chartData = data
                .map(lp => [lp.name, lp.launch_attempts])
                .sort((a, b) => b[1] - a[1]); // Ordenamos de mayor a menor actividad

           
            Highcharts.chart('spacex-pyramid-container', {
                chart: {
                    type: 'pyramid', // REQUISITO: No es line, ni bar, ni map
                    backgroundColor: '#fdfdfd'
                },
                title: {
                    text: 'Actividad por Base de Lanzamiento SpaceX'
                },
                subtitle: {
                    text: 'Basado en el número total de intentos de lanzamiento'
                },
                plotOptions: {
                    series: {
                        dataLabels: {
                            enabled: true,
                            format: '<b>{point.name}</b> ({point.y})',
                            softConnector: true
                        },
                        center: ['40%', '50%'],
                        width: '70%'
                    }
                },
                legend: {
                    enabled: false
                },
                series: [{
                    name: 'Intentos de lanzamiento',
                    data: chartData
                }]
            });

            isLoading = false;
        } catch (error) {
            console.error("Error cargando la integración de SpaceX:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>SpaceX Launchpads</h1>
    
    

    <!-- Contenedor del Gráfico -->
    <div id="spacex-pyramid-container"></div>

</div>

<style>
    #spacex-pyramid-container {
        height: 500px;
        width: 100%;
        max-width: 800px;
        margin: 20px auto;
        border-radius: 8px;
        box-shadow: 0 4px 15px rgba(0,0,0,0.1);
    }
</style>