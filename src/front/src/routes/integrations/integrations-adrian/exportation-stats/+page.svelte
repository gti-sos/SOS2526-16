<script>
    import { onMount } from "svelte";
    import { browser } from '$app/environment';

    let chartContainer;
    let Highcharts;
    let isLoading = true;
    let integratedYears = [];

    async function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) return resolve();
            const script = document.createElement("script");
            script.src = src; script.onload = resolve; script.onerror = reject;
            document.head.appendChild(script);
        });
    }

    onMount(async () => {
        if (!browser) return;
        try {
            // Cargar librerías
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            Highcharts = window.Highcharts;

            // Fetch
            // Carga de datos
            await fetch("/api/v1/global-ev-stock-volumes/loadInitialData").catch(() => {});
            
            const resStock = await fetch("/api/v1/global-ev-stock-volumes");
            const resWeapons = await fetch("/api/v1/proxy/exportation-stats");

            if (!resStock.ok || !resWeapons.ok) {
                throw new Error("Una de las APIs no ha respondido correctamente");
            }

            const stockData = await resStock.json();
            const weaponsData = await resWeapons.json();

            // Integramos los datos por año
            const allYears = [...new Set([
                ...stockData.map(s => Number(s.year)),
                ...weaponsData.map(w => Number(w.year_of_order))
            ])].filter(y => !isNaN(y)).sort((a, b) => b - a);

            integratedYears = allYears.map(year => {
                const totalStock = stockData
                    .filter(s => Number(s.year) === year)
                    .reduce((acc, curr) => acc + (Number(curr.ev_stock) || 0), 0);

                const totalWeapons = weaponsData
                    .filter(w => Number(w.year_of_order) === year)
                    .reduce((acc, curr) => acc + (Number(curr.tiv_total_order) || 0), 0);

                return {
                    name: "Año " + year,
                    y: totalWeapons,
                    year: year,
                    evStock: totalStock
                };
            }).filter(item => item.y > 0 || item.evStock > 0);

            // Cargamos gráfico
            if (Highcharts && chartContainer && integratedYears.length > 0) {
                Highcharts.chart(chartContainer, {
                    chart: { type: "pie" },
                    title: { text: "Integración: Armamento vs Stock EV" },
                    tooltip: {
                        useHTML: true,
                        pointFormat: '<b>Armas:</b> {point.y} TIV<br/><span style="color:green"><b>Stock EV:</b> {point.evStock} uds</span>'
                    },
                    plotOptions: {
                        pie: {
                            innerSize: "60%", 
                            dataLabels: { enabled: true, format: '<b>{point.name}</b><br>EV: {point.evStock}' }
                        }
                    },
                    series: [{ name: "Datos", colorByPoint: true, data: integratedYears }]
                });
            }
            isLoading = false;
        } catch (error) {
            console.error("Fallo detallado:", error);
            isLoading = false;
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>Comparativa Temporal</h1>
    

    <div bind:this={chartContainer} style="width: 100%; max-width: 900px; height: 500px; margin: 0 auto; border: 1px solid #ddd; border-radius: 10px; background: white;"></div>
</div>