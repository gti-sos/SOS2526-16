<script>
    import { onMount } from "svelte";

    let Highcharts;

    // Función de carga dinámica de scripts (estilo de tu compañero)
    function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) return resolve();
            const script = document.createElement("script");
            script.src = src;
            script.onload = resolve;
            script.onerror = reject;
            document.head.appendChild(script);
        });
    }

    onMount(async () => {
        try {
            // 1. CARGAR LIBRERÍAS
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            Highcharts = window.Highcharts;

            // 2. CARGAR DATOS DE LA API
            const res = await fetch("https://sos2526-13.onrender.com/api/v2/exportations-stats");
            const data = await res.json();

            // 3. PREPARAR DATOS PARA EL GRÁFICO DE DONUT
            // Mapeamos para tener: { name: "País Receptor", y: valor_del_pedido }
            const chartData = data.map(d => ({
                name: d.recipient,
                y: Number(d.tiv_total_order),
                supplier: d.supplier,   // Guardamos extra para el tooltip
                year: d.year_of_order   // Guardamos extra para el tooltip
            }));

            // 4. RENDERIZAR WIDGET
            Highcharts.chart("container-orders", {
                chart: {
                    type: "pie" 
                },
                title: {
                    text: "Distribución de Pedidos Internacionales (TIV)"
                },
                subtitle: {
                    text: "Porcentaje del total de pedidos por país receptor"
                },
                tooltip: {
                    useHTML: true,
                    pointFormat: '<b>Proveedor:</b> {point.supplier}<br/>' +
                                 '<b>Año:</b> {point.year}<br/>' +
                                 '<b>Valor:</b> {point.y} TIV ({point.percentage:.1f}%)'
                },
                plotOptions: {
                    pie: {
                        innerSize: "60%", 
                        allowPointSelect: true,
                        cursor: "pointer",
                        dataLabels: {
                            enabled: true,
                            format: "<b>{point.name}</b>: {point.y}"
                        }
                    }
                },
                series: [{
                    name: "Valor del pedido",
                    colorByPoint: true,
                    data: chartData
                }]
            });
            isLoading = false;
        } catch (error) {
            console.error("Error cargando la integración de pedidos:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px;">
    <h1>Integración SOS: Pedidos Internacionales</h1>
    <div id="container-orders" style="width: 100%; max-width: 800px; height: 500px; margin: 0 auto;"></div>
</div>