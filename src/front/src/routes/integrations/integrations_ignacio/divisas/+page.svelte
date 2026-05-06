<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let chartContainer;
    let cargando = $state(true);
    let errorMensaje = $state('');

    // Mapeo de tus países a su moneda
    const monedas = {
        'australia': 'AUD',
        'usa': 'USD',
        'united kingdom': 'GBP',
        'canada': 'CAD',
        'china': 'CNY'
    };

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

            // 2. Fetch de tus datos (EV) y de la API de Divisas (Frankfurter)
            const resEV = await fetch('/api/v1/global-ev-sales');
            const resDivisas = await fetch('/api/v1/global-ev-sales/proxy/divisas');

            if (!resEV.ok || !resDivisas.ok) throw new Error("Error al conectar con las APIs");

            const misDatos = await resEV.json();
            const dataDivisas = await resDivisas.json();

            const labels = [];
            const valoresEV = [];
            const valoresDivisa = [];

            // 3. Cruzar datos: ¿Cuánto vale 1€ en ese país comparado con tus ventas?
            // Filtramos por un año reciente (2021)
            const misDatos2021 = misDatos.filter(d => Number(d.year) === 2021);

            misDatos2021.forEach(miDato => {
                const codigoMoneda = monedas[miDato.region.toLowerCase().trim()];
                const tasaCambio = dataDivisas.rates[codigoMoneda];

                if (tasaCambio) {
                    labels.push(`${miDato.region} (${codigoMoneda})`);
                    valoresEV.push(miDato.value);
                    valoresDivisa.push(tasaCambio);
                }
            });

            if (labels.length === 0) {
                errorMensaje = "No hay países comunes con moneda propia (evitamos el Euro para comparar).";
                cargando = false;
                return;
            }

            // 4. Crear la gráfica Doughnut (PERMITIDA)
            new window.Chart(chartContainer, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [
                        {
                            label: 'Ventas EV',
                            data: valoresEV,
                            backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56', '#4BC9C2', '#9966FF'],
                        },
                        {
                            label: 'Valor de 1€ en moneda local',
                            data: valoresDivisa,
                            backgroundColor: ['#FF6384AA', '#36A2EBAA', '#FFCE56AA', '#4BC9C2AA', '#9966FFAA'],
                            borderWidth: 5
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'bottom' },
                        title: { display: true, text: 'Ventas EV vs Valor del Euro' }
                    }
                }
            });

            cargando = false;
        } catch (e) {
            console.error(e);
            errorMensaje = "Error al cargar la integración de divisas.";
            cargando = false;
        }
    });
</script>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2 style="text-align: center;">Integración: Ventas EV y Valor de Divisa</h2>
    <p style="text-align: center; color: #666;">
        Cruzando ventas con el tipo de cambio actual frente al Euro (<strong>Frankfurter API</strong>).
    </p>

    <div style="height: 500px; width: 100%; background: white; padding: 20px; border-radius: 10px; border: 1px solid #ddd; position: relative;">
        {#if cargando}
            <div style="text-align: center; padding-top: 100px;">Consultando mercado de divisas...</div>
        {/if}
        <canvas bind:this={chartContainer}></canvas>
    </div>
</main>