<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let chartContainer;
    let cargando = $state(true);
    let errorMensaje = $state('');

    onMount(async () => {
        if (!browser) return;

        try {
            // 1. Cargar la librería Chart.js
            await new Promise((resolve) => {
                if (window.Chart) return resolve();
                const script = document.createElement('script');
                script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
                script.onload = resolve;
                document.head.appendChild(script);
            });

            // 2. Fetch directo a Gutendex (Libre de CORS)
            const res = await fetch('https://gutendex.com/books/');
            
            if (!res.ok) throw new Error("La biblioteca digital no responde.");
            
            const datos = await res.json();

            // 3. Extraer el top 5 de libros más populares
            const topLibros = datos.results.slice(0, 5);

            // Recortamos los títulos si son muy largos para que la leyenda se vea bien
            const labels = topLibros.map(l => 
                l.title.length > 25 ? l.title.substring(0, 25) + '...' : l.title
            );
            const descargas = topLibros.map(l => l.download_count);

            // 4. Crear Gráfica de Área Polar (Chart.js)
            new window.Chart(chartContainer, {
                type: 'polarArea', // Tipo legal y muy vistoso
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Número de Descargas',
                        data: descargas,
                        backgroundColor: [
                            'rgba(255, 99, 132, 0.6)',
                            'rgba(54, 162, 235, 0.6)',
                            'rgba(255, 206, 86, 0.6)',
                            'rgba(75, 192, 192, 0.6)',
                            'rgba(153, 102, 255, 0.6)'
                        ],
                        borderColor: '#ffffff',
                        borderWidth: 2
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { position: 'right' },
                        title: {
                            display: true,
                            text: 'Top 5 Libros de Dominio Público (Proyecto Gutenberg)'
                        }
                    }
                }
            });

            cargando = false;
        } catch (e) {
            console.error(e);
            errorMensaje = "Fallo al conectar con la API de libros.";
            cargando = false;
        }
    });
</script>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2 style="text-align: center;">Uso de API Externa: Gutendex</h2>
    <p style="text-align: center; color: #666;">
        Análisis de descargas de literatura clásica (Uso directo sin proxy).
    </p>

    {#if errorMensaje}
        <p style="color: #d9534f; text-align: center; font-weight: bold;">{errorMensaje}</p>
    {/if}

    <div style="height: 500px; width: 100%; background: #fefefe; border-radius: 8px; border: 1px solid #e0e0e0; padding: 20px; box-shadow: 0 2px 4px rgba(0,0,0,0.05);">
        {#if cargando}
            <div style="text-align: center; padding-top: 150px; font-style: italic; color: #888;">
                Consultando los archivos de la biblioteca...
            </div>
        {/if}
        <canvas bind:this={chartContainer}></canvas>
    </div>

    <div style="margin-top: 30px; text-align: center;">
        <a href="/analytics/global-ev-sales/integrations" 
           style="background-color: #007bff; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;">
           ← Volver al Menú
        </a>
    </div>
</main>