<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let carMakes = [];
    let totalMakes = 0;

    // Función para cargar C3 desde internet (evita el error 500 del servidor)
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
            // 1. CARGAR LIBRERÍAS EXTERNAS
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/d3/5.16.0/d3.min.js");
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.js");
            
            // Inyectar el CSS de C3
            const link = document.createElement('link');
            link.rel = 'stylesheet';
            link.href = 'https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.css';
            document.head.appendChild(link);

            const c3 = window.c3;

            // 2. PETICIÓN FETCH A TU PROPIO PROXY (Sin claves ni headers aquí)
            const res = await fetch('/api/v1/proxy/cars');
            const result = await res.json();
            
            carMakes = result.data || [];
            totalMakes = carMakes.length;

            // 3. GENERACIÓN DEL WIDGET (Tipo Gauge - Medidor)
            c3.generate({
                bindto: '#car-gauge-chart',
                data: {
                    columns: [['Marcas Totales', totalMakes]],
                    type: 'gauge' // REQUISITO: Tipo distinto a line
                },
                gauge: {
                    label: { format: (v) => v + " marcas" },
                    min: 0, 
                    max: 100, 
                    units: ' marcas'
                },
                color: {
                    pattern: ['#FF0000', '#F97600', '#F6C600', '#60B044'],
                    threshold: { values: [30, 60, 90, 100] }
                }
            });
        } catch (error) {
            console.error("Error cargando la Car API:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px;">
    <h1>Integración Car API (Vía Proxy + C3.js)</h1>
</div>