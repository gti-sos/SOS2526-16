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

            // 2. Hacemos fetch a tu API y a la del compañero
            const resMisDatos = await fetch('/api/v1/global-ev-sales');
            const resMilitares = await fetch('/api/v1/global-ev-sales/proxy/military-stats');

            if (!resMisDatos.ok || !resMilitares.ok) {
                throw new Error('Error al conectar con una de las APIs');
            }

            const misDatos = await resMisDatos.json();
            const datosMilitares = await resMilitares.json();

            // 3. Cruzar los datos (Usamos el año 2022 según el JSON)
            const anioComun = 2022; 
            
            const misDatosFiltrados = misDatos.filter(d => Number(d.year) === anioComun);
            const militaresFiltrados = datosMilitares.filter(d => Number(d.year) === anioComun);

            const paisesComunes = [];
            const valoresEV = [];
            const valoresMilitares = [];

            // Comparamos y unimos los que coincidan en nombre de país
            misDatosFiltrados.forEach(miDato => {
                const pais = miDato.region.toLowerCase();
                // Usamos d.country tal y como viene en su JSON
                const matchMilitar = militaresFiltrados.find(d => d.country.toLowerCase() === pais);

                if (matchMilitar) {
                    paisesComunes.push(miDato.region);
                    valoresEV.push(miDato.value);
                    // Usamos d.milex_total para el gasto
                    valoresMilitares.push(matchMilitar.milex_total); 
                }
            });

            if (paisesComunes.length === 0) {
                errorMensaje = `No se han encontrado países en común para el año ${anioComun}.`;
                cargando = false;
                return;
            }
            cargando = false;

            // 4. Dibujar la gráfica Radar con Chart.js
            new window.Chart(chartContainer, {
                type: 'radar',
                data: {
                    labels: paisesComunes,
                    datasets: [
                        {
                            label: 'Ventas EV (Tus datos)',
                            data: valoresEV,
                            backgroundColor: 'rgba(51, 181, 229, 0.2)',
                            borderColor: 'rgba(51, 181, 229, 1)',
                            pointBackgroundColor: 'rgba(51, 181, 229, 1)',
                            borderWidth: 2
                        },
                        {
                            label: 'Gasto Militar (milex_total)',
                            data: valoresMilitares,
                            backgroundColor: 'rgba(255, 68, 68, 0.2)',
                            borderColor: 'rgba(255, 68, 68, 1)',
                            pointBackgroundColor: 'rgba(255, 68, 68, 1)',
                            borderWidth: 2
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        r: {
                            angleLines: { display: true },
                            suggestedMin: 0
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
    <title>Integración - Military Stats</title>
</svelte:head>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; text-align: center; font-family: sans-serif;">
    <h2>Correlación: Military Stats vs Ventas de EV</h2>
    <p style="color: #666; margin-bottom: 30px;">Integración de <strong>global-ev-sales</strong> con la API de <strong>Military Stats</strong> mediante Proxy</p>

    {#if errorMensaje}
        <div style="padding: 15px; background-color: #ff4444; color: white; border-radius: 5px; margin-bottom: 20px;">
            {errorMensaje}
        </div>
    {/if}

    <div style="position: relative; height: 500px; width: 100%; border: 1px solid #ddd; border-radius: 8px; padding: 10px; background-color: #f9f9f9;">
        
        {#if cargando}
            <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 18px; color: #00C851; font-weight: bold;">
               
            </div>
        {/if}

        <canvas bind:this={chartContainer}></canvas>
    </div>

    <div style="margin-top: 30px;">
        <a href="/integrations" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px; font-weight: bold;">← Volver a Integraciones</a>
    </div>
</main>