<script>
    import { onMount } from "svelte";
    import { browser } from '$app/environment';

    let container;
    let Highcharts;
    let isLoading = true;
    let errorMsg = ""; // Para avisar si hay un error 429 o de red

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
            // Carga de módulos de la librería
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/sankey.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            Highcharts = window.Highcharts;

            // fetch API compañero
            // Cargamos datos iniciales mi API
            await fetch("/api/v1/global-ev-stock-volumes/loadInitialData");

            const resStock = await fetch("/api/v1/global-ev-stock-volumes");
            const resAlcohol = await fetch("/api/v1/proxy/social-drinking-behaviors");

            // Comprobar si los servidores responden bien (evita 429/404). DEPURACIÓN
            if (!resStock.ok || !resAlcohol.ok) {
                errorMsg = `El servidor del socio está saturado (Error ${resAlcohol.status}). Por favor, espera 1 minuto y recarga.`;
                isLoading = false;
                return;
            }

            const stockData = await resStock.json();
            const alcoholData = await resAlcohol.json();

            //Comprobación extra(ver si recibimos lista). DEPURACIÓN
            if (!Array.isArray(stockData) || !Array.isArray(alcoholData)) {
                errorMsg = "Los datos recibidos no tienen el formato correcto.";
                isLoading = false;
                return;
            }

            // Carga de datos
            const commonCountries = ["Canada", "Japan", "China"];
            let sankeyData = [];

            commonCountries.forEach(country => {
                const s = stockData.find(d => d.region_country?.toLowerCase().trim() === country.toLowerCase());
                const a = alcoholData.find(d => d.country?.toLowerCase().trim() === country.toLowerCase());

                // Solo añadimos si hay datos
                if (s || a) {
                    // Rama 1: EV vehicles
                    sankeyData.push([country, 'Stock Vehículos Eléctricos', s ? (s.ev_stock / 50) : 150]);
                    
                    if (a) {
                        // Rama 2: Alcohol
                        const scale = 20;
                        sankeyData.push([country, 'Consumo Alcohol', Number(a.total_liter) * scale]);
                        sankeyData.push(['Consumo Alcohol', 'Cerveza', Number(a.beer_share) * scale]);
                        sankeyData.push(['Consumo Alcohol', 'Vino', Number(a.wine_share) * scale]);
                        sankeyData.push(['Consumo Alcohol', 'Espirituosos', Number(a.spirit_share) * scale]);
                    }
                }
            });

            // Visualización del gráfico
            if (sankeyData.length > 0) {
                Highcharts.chart(container, {
                    title: { text: 'Estadísticas consumo alcohol y Stock EV' },
                    
                    series: [{
                        keys: ['from', 'to', 'weight'],
                        data: sankeyData,
                        type: 'sankey',
                        name: 'Flujo de datos'
                    }]
                });
            } else {
                errorMsg = "No se han encontrado coincidencias entre las dos bases de datos.";
            }
            isLoading = false;
        } catch (error) {
            console.error("Error en Sankey:", error);
            errorMsg = "Ha ocurrido un error inesperado al procesar la gráfica.";
            isLoading = false;
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>Integración EV Stock y Consumo de alcochol mediante  Sankey Diagram</h1>

    <div bind:this={container} style="width: 100%; max-width: 1000px; height: 600px; margin: 20px auto; background: white; border: 1px solid #ddd; border-radius: 8px;"></div>

    
</div>