<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let temperature = 0;
    let isLoading = true;

    // Función para cargar librerías externas (D3 y C3)
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
            // 1. CARGAR LIBRERÍAS C3 Y D3 (Evita errores de SvelteKit)
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/d3/5.16.0/d3.min.js");
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.js");
            
            // Inyectar el CSS de C3
            const link = document.createElement('link');
            link.rel = 'stylesheet';
            link.href = 'https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.css';
            document.head.appendChild(link);

            const c3 = window.c3;

            // 2. PETICIÓN FETCH DIRECTA (API con CORS Abierto)
            // Consultamos el tiempo actual en Sevilla (puedes cambiar lat/lon)
            const url = 'https://api.open-meteo.com/v1/forecast?latitude=37.38&longitude=-5.98&current_weather=true';
            const res = await fetch(url);
            const json = await res.json();
            
            // Extraemos la temperatura actual
            temperature = json.current_weather.temperature;

            // 3. GENERACIÓN DEL GRÁFICO GAUGE (C3.js)
            c3.generate({
                bindto: '#weather-gauge',
                data: {
                    columns: [
                        ['Temperatura', temperature]
                    ],
                    type: 'gauge' // REQUISITO: No es line
                },
                gauge: {
                    label: {
                        format: (value) => value + "°C",
                        show: true
                    },
                    min: -10,
                    max: 50,
                    units: ' °C'
                },
                color: {
                    pattern: ['#3498db', '#f1c40f', '#e67e22', '#e74c3c'], // De frío a calor
                    threshold: {
                        values: [10, 25, 35, 45]
                    }
                },
                size: {
                    height: 250
                }
            });
            
            isLoading = false;
        } catch (error) {
            console.error("Error en la integración climática:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>Integración Externa: Clima Local (Open-Meteo)</h1>
    <p>Temperatura actual en tiempo real obtenida mediante fetch directo.</p>

    {#if isLoading}
        <div style="padding: 20px;">
            <p>Obteniendo datos meteorológicos...</p>
            <progress></progress>
        </div>
    {/if}

    <!-- Contenedor de la gráfica -->
    <div id="weather-gauge"></div>

    <div style="margin-top: 30px; border-top: 1px solid #eee; padding-top: 15px;">
        <h3>Datos Textuales (Requisito i)</h3>
        <p><b>Ubicación:</b> Sevilla, España</p>
        <p><b>Temperatura:</b> {temperature} °C</p>
        <p><i>Esta visualización no usa proxy al tener la API el CORS abierto.</i></p>
    </div>
</div>

<style>
    #weather-gauge {
        width: 100%;
        max-width: 500px;
        margin: 0 auto;
    }
</style>