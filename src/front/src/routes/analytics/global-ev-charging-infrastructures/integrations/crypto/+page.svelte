<script>
    import { onMount } from "svelte";

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

    onMount(async () => {
        await loadScript("https://code.highcharts.com/highcharts.js");

        const res = await fetch("/api/v1/global-ev-charging-infrastructures/proxy/crypto");
        const data = await res.json();

        console.log("CRYPTO DATA:", data);

        // ordenar por volumen y coger top 10
        const top10 = data
            .sort((a, b) => Number(b.volume) - Number(a.volume))
            .slice(0, 10);

        const categories = top10.map(d => d.symbol);
        const seriesData = top10.map(d => ({
            y: Number(d.volume),
            custom: d
        }));

        Highcharts.chart("container", {
            chart: {
                type: "column"
            },

            title: {
                text: "Top 10 Cryptos by Volume (24h)"
            },

            xAxis: {
                categories
            },

            yAxis: {
                title: {
                    text: "Volume"
                }
            },

            tooltip: {
                useHTML: true,
                formatter: function () {
                    const d = this.point.custom;

                    return `
                        <b>${d.symbol}</b><br/><br/>

                        Price: ${d.lastPrice}<br/>
                        Change: ${d.priceChange} (${d.priceChangePercent}%)<br/>
                        High: ${d.highPrice}<br/>
                        Low: ${d.lowPrice}<br/>
                        Volume: ${d.volume}<br/>
                        Trades: ${d.count}
                    `;
                }
            },

            series: [{
                name: "Volume",
                data: seriesData
            }]
        });
    });
</script>

<h1>Crypto Dashboard</h1>

<div id="container"></div>

<style>
    h1 {
        text-align: center;
        font-family: Arial, sans-serif;
    }

    #container {
        width: 100%;
        height: 500px;
        margin: 0 auto;
    }
</style>