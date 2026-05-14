<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    // ref al div donde se va a pintar la grafica
    let chartContainer;
    let datos = [];
    let errorMensaje = '';

    onMount(async () => {
        // para que no reviente al cargar en el servidor 
        if (!browser) return;

        try {
            // fuerzo carga de datos iniciales por si la bd esta vacia
            await fetch('/api/v1/global-ev-sales/loadInitialData');
            const res = await fetch('/api/v1/global-ev-sales');
            
            if (!res.ok) throw new Error('No se pudieron obtener los datos de la API');
            datos = await res.json();

            if (datos.length === 0) {
                errorMensaje = 'No hay datos disponibles para mostrar la gráfica.';
                return;
            }

            // objeto vacio pa agrupar las lineas de la grafica segun el tipo de motor
            const seriesData = {};
            
            //crea una lista nueva para cada tipo de motor la primera vez que entran y agrupa los datos sin dar error
            datos.forEach(d => {
                const motor = d.powertrain || 'Desconocido';
                if (!seriesData[motor]) {
                    seriesData[motor] = [];
                }
                
                // highcharts bubble pide la info con esta estructura exacta de letras
                seriesData[motor].push({
                    name: d.region,
                    x: d.year,
                    y: d.economic_impact,
                    z: d.value, // la z es lo q define si la burbuja se ve mas grande o pequeña
                    categoria: d.category,
                    modo: d.mode
                });
            });

            // como highcharts no traga objetos directos lo paso a un array mapeandolo
            const seriesArray = Object.keys(seriesData).map(motorName => {
                return {
                    name: `Motor: ${motorName}`,
                    data: seriesData[motorName]
                };
            });

            // pillo la libreria del navegador que he metido luego en el head
            const Highcharts = window.Highcharts;

            // le paso el contenedor y el tocho de configuracion visual
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
                    labels: {
                        format: '{value}' // pa q no ponga los años con puntos de miles tipo 2.021
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
                    useHTML: true, // activo esto pa meterle una tabla html cuando pasas el raton
                    headerFormat: '<table style="min-width: 150px;">',
                    pointFormat: '<tr><th colspan="2" style="border-bottom: 1px solid #ccc; padding-bottom: 5px;"><h3>{point.name}</h3></th></tr>' +
                        '<tr><th style="text-align: left; padding-top: 5px;">Año:</th><td style="text-align: right; padding-top: 5px;">{point.x}</td></tr>' +
                        '<tr><th style="text-align: left;">Impacto Económico:</th><td style="text-align: right;">{point.y}</td></tr>' +
                        '<tr><th style="text-align: left;">Volumen Ventas:</th><td style="text-align: right;">{point.z}</td></tr>' +
                        '<tr><th style="text-align: left;">Modo:</th><td style="text-align: right;">{point.modo}</td></tr>' +
                        '<tr><th style="text-align: left;">Categoría:</th><td style="text-align: right;">{point.categoria}</td></tr>',
                    footerFormat: '</table>',
                    followPointer: true
                },
                plotOptions: {
                    bubble: {
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
    <script src="https://code.highcharts.com/highcharts.js"></script>
    <script src="https://code.highcharts.com/highcharts-more.js"></script>
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