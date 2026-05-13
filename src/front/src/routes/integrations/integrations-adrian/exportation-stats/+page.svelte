<script>
    import { onMount } from "svelte";
    import { browser } from '$app/environment';

    let chartContainer;
    let Highcharts;
    let isLoading = true;
    let integratedYears = []; // Datos agrupados por año para la tabla

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
            // 1. CARGAR LIBRERÍAS
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            Highcharts = window.Highcharts;

            // 2. FETCH PARALELO DE LAS APIS
            // Cargamos tus datos iniciales para asegurar que hay contenido
            await fetch("/api/v1/global-ev-stock-volumes/loadInitialData");
            
            const [resStock, resWeapons] = await Promise.all([
                fetch("/api/v1/global-ev-stock-volumes"),
                fetch("/api/v1/proxy/exportation-stats")
            ]);

            const stockData = await resStock.json();
            const weaponsData = await resWeapons.json();

            // 3. INTEGRACIÓN POR AÑO (Lógica de cruce)
            // Obtenemos todos los años únicos de ambas APIs
            const allYears = [...new Set([
                ...stockData.map(s => Number(s.year)),
                ...weaponsData.map(w => Number(w.year_of_order))
            ])].sort((a, b) => b - a); // Ordenar de más reciente a más antiguo

            integratedYears = allYears.map(year => {
                // Sumamos todos tus vehículos de ese año en todos los países
                const totalStock = stockData
                    .filter(s => Number(s.year) === year)
                    .reduce((acc, curr) => acc + curr.ev_stock, 0);

                // Sumamos todos los pedidos de armas del socio de ese año
                const totalWeapons = weaponsData
                    .filter(w => Number(w.year_of_order) === year)
                    .reduce((acc, curr) => acc + Number(curr.tiv_total_order), 0);

                return {
                    name: "Año " + year,
                    y: totalWeapons, // El tamaño de la porción es el volumen de armas
                    year: year,
                    evStock: totalStock // Guardamos el valor de tu API
                };
            }).filter(item => item.y > 0 || item.evStock > 0); // Solo mostrar años con datos

            // 4. RENDERIZAR GRÁFICA (Donut)
            Highcharts.chart(chartContainer, {
                chart: { type: "pie" },
                title: { text: "Pedidos armamento militar vs Stock EV" },
                subtitle: { text: "Comparativa del volumen total mundial por años" },
                tooltip: {
                    useHTML: true,
                    pointFormat: '<b>Año:</b> {point.year}<br/>' +
                                 '<b>Pedidos Armas:</b> {point.y} TIV<br/>' +
                                 '<span style="color:#2ecc71"><b>Total Stock EV:</b> {point.evStock} uds</span>'
                },
                plotOptions: {
                    pie: {
                        innerSize: "60%", 
                        dataLabels: {
                            enabled: true,
                            // Mostramos el año y TU dato de stock en la etiqueta
                            format: '<b>{point.name}</b><br><span style="color:#27ae60">Stock: {point.evStock}</span>'
                        }
                    }
                },
                series: [{
                    name: "Armamento (TIV)",
                    colorByPoint: true,
                    data: integratedYears
                }]
            });
            isLoading = false;
        } catch (error) {
            console.error("Error en la integración por años:", error);
            isLoading = false;
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>Comparativa Temporal</h1>
    
</div>

    <!-- Contenedor del Donut -->
    <div bind:this={chartContainer} style="width: 100%; max-width: 900px; height: 550px; margin: 0 auto; border: 1px solid #ddd; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);"></div>
