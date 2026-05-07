<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let chartContainer;
    let cargando = $state(true);
    let errorMensaje = $state('');

    onMount(async () => {
        if (!browser) return;

        try {
            // 1. Cargar Chart.js
            await new Promise((resolve) => {
                if (window.Chart) return resolve();
                const script = document.createElement('script');
                script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
                script.onload = resolve;
                document.head.appendChild(script);
            });

            // 2. Fetch de ambas APIs
            const resEV = await fetch('/api/v1/global-ev-sales');
            const resPoblacion = await fetch('https://restcountries.com/v3.1/all?fields=name,population');

            if (!resEV.ok || !resPoblacion.ok) throw new Error("Error al obtener los datos");

            const misDatos = await resEV.json();
            const paisesExternos = await resPoblacion.json();

            // 3. Filtrar y Cruzar
            // Usamos el año 2021 como ejemplo para tener más variedad
            const anio = 2021;
            const misDatosFiltrados = misDatos.filter(d => Number(d.year) === anio);

            const labels = [];
            const ratios = [];

            misDatosFiltrados.forEach(miDato => {
                const match = paisesExternos.find(p => 
                    p.name.common.toLowerCase() === miDato.region.toLowerCase() ||
                    p.name.official.toLowerCase() === miDato.region.toLowerCase()
                );

                if (match && match.population > 0) {
                    labels.push(miDato.region);
                    // Calculamos ventas por cada millón de habitantes
                    const ratio = (miDato.value / (match.population / 1000000)).toFixed(2);
                    ratios.push(ratio);
                }
            });

            if (labels.length === 0) {
                errorMensaje = "No se encontraron países coincidentes para el cruce de datos.";
                cargando = false;
                return;
            }

            // 4. Crear Gráfica de Barras Horizontales (PERMITIDA)
            new window.Chart(chartContainer, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: `Ventas de EV por cada millón de habitantes (${anio})`,
                        data: ratios,
                        backgroundColor: 'rgba(54, 162, 235, 0.6)',
                        borderColor: 'rgb(54, 162, 235)',
                        borderWidth: 1
                    }]
                },
                options: {
                    indexAxis: 'y', // Hace que las barras sean horizontales
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'top' },
                        tooltip: {
                            callbacks: {
                                label: (context) => ` ${context.raw} unidades / millón hab.`
                            }
                        }
                    }
                }
            });

            cargando = false;
        } catch (e) {
            console.error(e);
            errorMensaje = "Ocurrió un error al generar la integración de población.";
            cargando = false;
        }
    });
</script>

<main style="max-width: 1000px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2 style="text-align: center;">Penetración de Vehículos Eléctricos por Población</h2>
    <p style="text-align: center; color: #666;">
        Integración de datos propios con la API externa <strong>RestCountries</strong>.
    </p>

    {#if errorMensaje}
        <div style="color: red; text-align: center; padding: 20px;">{errorMensaje}</div>
    {/if}

    <div style="height: 600px; width: 100%; background: white; padding: 20px; border-radius: 10px; border: 1px solid #eee; position: relative;">
        {#if cargando}
            <div style="text-align: center; padding-top: 100px;">Calculando ratios poblacionales...</div>
        {/if}
        <canvas bind:this={chartContainer}></canvas>
    </div>

    <div style="margin-top: 30px; text-align: center;">
        <a href="/analytics/global-ev-sales/integrations" 
           style="padding: 10px 20px; background: #eee; color: #333; text-decoration: none; border-radius: 5px;">
           Volver al índice
        </a>
    </div>
</main>