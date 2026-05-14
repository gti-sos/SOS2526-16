<script>
    import { onMount } from "svelte";

    // Función para cargar scripts de forma dinámica
    function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) {
                resolve();
                return;
            }

            const script = document.createElement("script");
            script.src = src;
            script.onload = resolve;
            script.onerror = reject;
            document.head.appendChild(script);
        });
    }

    // Cargar Highcharts y luego obtener datos y crear el gráfico
    onMount(async () => {
        await loadScript("https://code.highcharts.com/highcharts.js");
        await loadScript("https://code.highcharts.com/highcharts-more.js");

        const res = await fetch("/api/v1/global-ev-charging-infrastructures/proxy/crypto-exchanges");

        if (!res.ok) {
            console.error("Error al cargar exchanges:", res.status);
            return;
        }

        const data = await res.json();

        console.log("EXCHANGES DATA:", data);

        // Tomamos los top 20 exchanges por volumen, filtrando 
        // aquellos con volumen o pares activos igual a 0 para evitar
        //  mostrar burbujas sin datos
        const top20 = data
            .filter(d =>
                Number(d.volume_usd) > 0 &&
                Number(d.active_pairs) > 0
            )
            .sort((a, b) => Number(b.volume_usd) - Number(a.volume_usd))
            .slice(0, 20);

        const bubbleData = top20.map(d => ({
            name: d.name,
            x: Number(d.active_pairs),
            y: Number(d.volume_usd),
            z: Number(d.volume_usd),
            custom: d
        }));

        Highcharts.chart("container", {
            chart: {
                type: "bubble",
                zoomType: "xy"
            },

            title: {
                text: "Top 20 Crypto Exchanges by Volume"
            },

            subtitle: {
                text: "Bubble size represents USD volume"
            },

            xAxis: {
                title: {
                    text: "Active pairs"
                }
            },

            yAxis: {
                title: {
                    text: "Volume USD"
                }
            },

            tooltip: {
                useHTML: true,
                formatter: function () {
                    const d = this.point.custom;

                    return `
                        <b>${d.name}</b><br/><br/>
                        ID: ${d.id}<br/>
                        Name ID: ${d.name_id}<br/>
                        Country: ${d.country ?? "N/A"}<br/>
                        Volume USD: ${Number(d.volume_usd).toLocaleString()}<br/>
                        Active pairs: ${d.active_pairs}<br/>
                        URL: ${d.url ?? "N/A"}
                    `;
                }
            },

            plotOptions: {
                bubble: {
                    minSize: 8,
                    maxSize: 70,
                    dataLabels: {
                        enabled: true,
                        format: "{point.name}"
                    }
                }
            },

            series: [{
                name: "Crypto exchanges",
                data: bubbleData
            }]
        });
    });
</script>

<h1>Crypto Exchanges Bubble Chart</h1>

<div id="container"></div>
