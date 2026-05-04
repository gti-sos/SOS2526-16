<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let chartContainer;
    //Añadimos $state() para que Svelte actualice la pantalla
    let errorMensaje = $state('');
    let cargando = $state(true);

    onMount(async () => {
        if (!browser) return;

        try {
            // 1. Cargamos Chart.js dinámicamente
            await new Promise((resolve, reject) => {
                if (window.Chart) return resolve();
                const script = document.createElement('script');
                script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
                script.onload = resolve;
                script.onerror = reject;
                document.head.appendChild(script);
            });

            // 2. Cargamos datos iniciales propios por si Render se ha reiniciado
            await fetch('/api/v1/global-ev-sales/loadInitialData');

            // 3. Fetch a tu API y a la del compañero
            const resMisDatos = await fetch('/api/v1/global-ev-sales');
            
            const resFelicidad = await fetch('/api/v1/proxy/happiness-indices');

            if (!resMisDatos.ok || !resFelicidad.ok) {
                throw new Error('Error al conectar con una de las APIs. Puede que su servidor esté caído.');
            }

            const misDatos = await resMisDatos.json();
            const datosFelicidad = await resFelicidad.json();

            // 4. Cruzar los datos (Usaremos el año 2022)
            const anioComun = 2022; 
            
            const misDatosFiltrados = misDatos.filter(d => Number(d.year) === anioComun);
            const felicidadFiltrada = datosFelicidad.filter(d => Number(d.year) === anioComun);

            const scatterData = [];

            // Buscamos coincidencias de país
            misDatosFiltrados.forEach(miDato => {
                const pais = miDato.region.toLowerCase();
                const matchFelicidad = felicidadFiltrada.find(d => d.country.toLowerCase() === pais);

                if (matchFelicidad) {
                    scatterData.push({
                        x: matchFelicidad.happiness_score, 
                        y: miDato.value,                   
                        country: miDato.region             
                    });
                }
            });

            if (scatterData.length === 0) {
                errorMensaje = `No se encontraron países en común para el año ${anioComun}.`;
                cargando = false;
                return;
            }

            // 5. Dibujar la gráfica
            new window.Chart(chartContainer, {
                type: 'scatter',
                data: {
                    datasets: [{
                        label: 'Felicidad vs Ventas EV',
                        data: scatterData,
                        backgroundColor: '#00C851',
                        borderColor: '#007E33',
                        pointRadius: 8,
                        pointHoverRadius: 12
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: 'Puntuación de Felicidad (Happiness Score)',
                                font: { size: 14, weight: 'bold' }
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Volumen de Ventas de EV',
                                font: { size: 14, weight: 'bold' }
                            },
                            beginAtZero: true
                        }
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const rawData = context.raw;
                                    return `${rawData.country} - Felicidad: ${rawData.x}, Ventas: ${rawData.y}`;
                                }
                            }
                        }
                    }
                }
            });

            cargando = false;

        } catch (error) {
            console.error("Error en la integración:", error);
            errorMensaje = "Ocurrió un error de red o de permisos (CORS) al cargar la integración.";
            cargando = false;
        }
    });
</script>

<svelte:head>
    <title>Integración - Happiness Stats</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; text-align: center; font-family: sans-serif;">
    <h2>Correlación: Felicidad Nacional vs Ventas de EV</h2>
    <p style="color: #666; margin-bottom: 30px;">Integración de <strong>global-ev-sales</strong> con la API de <strong>Felicidad Mundial</strong> mediante Proxy</p>

    {#if errorMensaje}
        <div style="padding: 15px; background-color: #ff4444; color: white; border-radius: 5px; margin-bottom: 20px;">
            {errorMensaje}
        </div>
    {/if}

    <div style="position: relative; height: 500px; width: 100%; border: 1px solid #ddd; border-radius: 8px; padding: 10px; background-color: #f9f9f9;">
        
        {#if cargando}
            <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 18px; color: #00C851; font-weight: bold;">
                Cruzando datos y generando gráfico...
            </div>
        {/if}

        <canvas bind:this={chartContainer}></canvas>
    </div>

    <div style="margin-top: 30px;">
        <a href="/integrations" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px; font-weight: bold;">← Volver a Integraciones</a>
    </div>
</main>
