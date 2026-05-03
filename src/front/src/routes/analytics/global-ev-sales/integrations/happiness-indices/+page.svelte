<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let chartContainer;
    let errorMensaje = '';
    let cargando = true;

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

            // 2. Fetch a tu API y a la del compañero
            const resMisDatos = await fetch('/api/v1/global-ev-sales');
            
            const resFelicidad = await fetch('/api/v2/happiness-indices');

            if (!resMisDatos.ok || !resFelicidad.ok) {
                throw new Error('Error al conectar con una de las APIs');
            }

            const misDatos = await resMisDatos.json();
            const datosFelicidad = await resFelicidad.json();

            // 3. Cruzar los datos (Usaremos el año 2022 que parece tener muchos registros)
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
                        x: matchFelicidad.happiness_score, // Eje X: Felicidad
                        y: miDato.value,                   // Eje Y: Ventas EV
                        country: miDato.region             // Guardamos el nombre para el tooltip
                    });
                }
            });

            if (scatterData.length === 0) {
                errorMensaje = `No se encontraron países en común para el año ${anioComun}.`;
                cargando = false;
                return;
            }

            // 4. Dibujar la gráfica Scatter con Chart.js
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
            errorMensaje = "Ocurrió un error al cargar la integración.";
            cargando = false;
        }
    });
</script>

<svelte:head>
    <title>Integración - Happiness Stats</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; text-align: center; font-family: sans-serif;">
    <h2>Correlación: Felicidad Nacional vs Ventas de EV</h2>
    <p style="color: #666; margin-bottom: 30px;">Integración de <strong>global-ev-sales</strong> con la API de <strong>Felicidad Mundial</strong></p>

    {#if cargando}
        <div style="padding: 20px; font-size: 18px; color: #00C851;">Cruzando datos y generando gráfico...</div>
    {/if}

    {#if errorMensaje}
        <div style="padding: 15px; background-color: #ff4444; color: white; border-radius: 5px; margin-bottom: 20px;">
            {errorMensaje}
        </div>
    {/if}

    <div style="position: relative; height: 500px; width: 100%; display: {cargando || errorMensaje ? 'none' : 'block'}; border: 1px solid #ddd; border-radius: 8px; padding: 10px;">
        <canvas bind:this={chartContainer}></canvas>
    </div>

    <div style="margin-top: 30px;">
        <a href="/integrations" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px; font-weight: bold;">← Volver a Integraciones</a>
    </div>
</main>