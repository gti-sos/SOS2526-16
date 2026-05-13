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
            // 1. CARGAR LIBRERÍAS
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/sankey.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            Highcharts = window.Highcharts;

            // 2. FETCH DE LAS APIS CON VALIDACIÓN
            // Cargamos tus datos iniciales para asegurar que hay contenido
            await fetch("/api/v1/global-ev-stock-volumes/loadInitialData");

            const resStock = await fetch("/api/v1/global-ev-stock-volumes");
            const resAlcohol = await fetch("/api/v1/proxy/social-drinking-behaviors");

            // 🟢 VALIDACIÓN 1: Comprobar si los servidores responden bien (evita el 429/404)
            if (!resStock.ok || !resAlcohol.ok) {
                errorMsg = `El servidor del socio está saturado (Error ${resAlcohol.status}). Por favor, espera 1 minuto y recarga.`;
                isLoading = false;
                return;
            }

            const stockData = await resStock.json();
            const alcoholData = await resAlcohol.json();

            // 🟢 VALIDACIÓN 2: Comprobar que recibimos listas (evita el crash de .find)
            if (!Array.isArray(stockData) || !Array.isArray(alcoholData)) {
                errorMsg = "Los datos recibidos no tienen el formato correcto.";
                isLoading = false;
                return;
            }

            // 3. PROCESAMIENTO PARA SANKEY
            const commonCountries = ["Canada", "Japan", "China"];
            let sankeyData = [];

            commonCountries.forEach(country => {
                const s = stockData.find(d => d.region_country?.toLowerCase().trim() === country.toLowerCase());
                const a = alcoholData.find(d => d.country?.toLowerCase().trim() === country.toLowerCase());

                // Solo añadimos si hay datos
                if (s || a) {
                    // Rama 1: Tus vehículos (Escalado para que se vea igual de grueso que el alcohol)
                    sankeyData.push([country, 'Stock Vehículos Eléctricos', s ? (s.ev_stock / 50) : 150]);
                    
                    if (a) {
                        // Rama 2: Su alcohol
                        const scale = 20;
                        sankeyData.push([country, 'Consumo Alcohol', Number(a.total_liter) * scale]);
                        sankeyData.push(['Consumo Alcohol', 'Cerveza', Number(a.beer_share) * scale]);
                        sankeyData.push(['Consumo Alcohol', 'Vino', Number(a.wine_share) * scale]);
                        sankeyData.push(['Consumo Alcohol', 'Espirituosos', Number(a.spirit_share) * scale]);
                    }
                }
            });

            // 4. RENDERIZADO
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
    <h1>Integración de Alto Nivel (Sankey Diagram)</h1>

    <div bind:this={container} style="width: 100%; max-width: 1000px; height: 600px; margin: 20px auto; background: white; border: 1px solid #ddd; border-radius: 8px;"></div>

    
</div>