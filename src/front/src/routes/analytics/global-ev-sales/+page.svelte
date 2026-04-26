<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';
    import Highcharts from 'highcharts';

    // Declaramos el contenedor donde se inyectará la gráfica
    let chartContainer;
    let datos = [];
    let errorMensaje = '';

    onMount(async () => {
        if (!browser) return;

        try {
            // Importar el módulo de burbujas dinámicamente para Svelte
            const module = await import('highcharts/highcharts-more');
            const initMore = module.default || module;
            initMore(Highcharts);

            // Cargar datos
            await fetch('/api/v1/global-ev-sales/loadInitialData');
            const res = await fetch('/api/v1/global-ev-sales');
            
            if (!res.ok) throw new Error('No se pudieron obtener los datos de la API');
            datos = await res.json();

            if (datos.length === 0) {
                errorMensaje = 'No hay datos disponibles para mostrar la gráfica.';
                return;
            }

            // Agrupar los datos por tipo de motor (Powertrain) para crear distintas series (colores)
            const seriesData = {};
            
            datos.forEach(d => {
                const motor = d.powertrain || 'Desconocido';
                if (!seriesData[motor]) {
                    seriesData[motor] = [];
                }
                
                // Formato exigido por Highcharts bubble: { name, x, y, z }
                seriesData[motor].push({
                    name: d.region,
                    x: d.year,
                    y: d.economic_impact,
                    z: d.value, // El tamaño de la burbuja depende del valor de ventas
                    categoria: d.category,
                    modo: d.mode
                });
            });

          
            
            const seriesArray = Object.keys(seriesData).map(motorName => {
                return {
                    name: `Motor: ${motorName}`,
                    data: seriesData[motorName]
                };
            });

            // Generar la gráfica
            Highcharts.chart(chartContainer, {
                chart: {
                    type: 'bubble', 
                    plotBorderWidth: 1,
                    zooming: {
                        type: 'xy'
                    }
                },
                legend: {
                    enabled: true
                },
                title: {
                    text: 'Análisis Multidimensional de Ventas de Vehículos Eléctricos'
                },
                subtitle: {
                    text: 'Eje X: Año | Eje Y: Impacto Económico | Tamaño: Volumen de Ventas'
                },
                xAxis: {
                    gridLineWidth: 1,
                    title: {
                        text: 'Año de Registro'
                    },
                    // Ajustamos las etiquetas para que no salgan números con comas (Ej. 2,021)
                    labels: {
                        format: '{value}'
                    }
                },
                yAxis: {
                    startOnTick: false,
                    endOnTick: false,
                    title: {
                        text: 'Impacto Económico'
                    },
                    labels: {
                        format: '{value}'
                    },
                    maxPadding: 0.2
                },
                tooltip: {
                    useHTML: true,
                    headerFormat: '<table>',
                    pointFormat: '<tr><th colspan="2"><h3>{point.name}</h3></th></tr>' +
                        '<tr><th>Año:</th><td>{point.x}</td></tr>' +
                        '<tr><th>Impacto Económico:</th><td>{point.y}</td></tr>' +
                        '<tr><th>Volumen Ventas:</th><td>{point.z}</td></tr>' +
                        '<tr><th>Modo:</th><td>{point.modo}</td></tr>' +
                        '<tr><th>Categoría:</th><td>{point.categoria}</td></tr>',
                    footerFormat: '</table>',
                    followPointer: true
                },
                plotOptions: {
                    bubble: {
                        // Hace que las burbujas sean un poco transparentes
                        marker: {
                            fillOpacity: 0.6
                        }
                    }
                },
                series: seriesArray
            });

        } catch (error) {
            console.error("Error al cargar la gráfica:", error);
            errorMensaje = "Ocurrió un error al cargar la gráfica.";
        }
    });
</script>

<svelte:head>
    <title>Analytics - Global EV Sales</title>
</svelte:head>

<main>
    <h2>Dashboard Individual - Ventas de Vehículos Eléctricos</h2>
    
    {#if errorMensaje}
        <div style="padding: 15px; background-color: #ff4444; color: white; border-radius: 5px; margin-bottom: 20px;">
            {errorMensaje}
        </div>
    {/if}

    <div bind:this={chartContainer} style="width: 100%; height: 600px; margin: 0 auto; box-shadow: 0 4px 8px rgba(0,0,0,0.1); border-radius: 8px;"></div>
    
    <div style="text-align: center; margin-top: 20px;">
        <a href="/global-ev-sales" style="padding: 10px 20px; background-color: #33b5e5; color: white; text-decoration: none; border-radius: 5px;">← Volver a la tabla de datos</a>
    </div>
</main>

<style>
    main {
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
        font-family: sans-serif;
    }
    h2 {
        color: #333;
        text-align: center;
        margin-bottom: 30px;
    }
</style>