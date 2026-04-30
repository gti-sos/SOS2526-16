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
        await loadScript("https://code.highcharts.com/highcharts-more.js");

        const res = await fetch("/api/v1/global-ev-charging-infrastructures/proxy/crypto");
        const data = await res.json();

        const top20 = data
            .filter(d =>
                Number(d.lastPrice) > 0 &&
                Number(d.volume) > 0 &&
                !isNaN(Number(d.priceChangePercent))
            )
            .sort((a, b) => Number(b.volume) - Number(a.volume))
            .slice(0, 20);

        const bubbleData = top20.map(d => ({
            name: d.symbol,
            x: Number(d.lastPrice),
            y: Number(d.priceChangePercent),
            z: Number(d.volume),
            custom: d
        }));

        Highcharts.chart("container", {
            chart: {
                type: "bubble",
                zoomType: "xy"
            },

            title: {
                text: "Top 20 Cryptos: Price, Change and Volume"
            },

            subtitle: {
                text: "Bubble size represents 24h volume"
            },

            xAxis: {
                title: {
                    text: "Last Price"
                }
            },

            yAxis: {
                title: {
                    text: "Price Change Percent (%)"
                },
                labels: {
                    format: "{value}%"
                }
            },

            tooltip: {
                useHTML: true,
                formatter: function () {
                    const d = this.point.custom;

                    return `
                        <b>${d.symbol}</b><br/><br/>
                        Last price: ${d.lastPrice}<br/>
                        Price change: ${d.priceChange}<br/>
                        Change percent: ${d.priceChangePercent}%<br/>
                        Weighted avg price: ${d.weightedAvgPrice}<br/>
                        Previous close: ${d.prevClosePrice}<br/>
                        High price: ${d.highPrice}<br/>
                        Low price: ${d.lowPrice}<br/>
                        Volume: ${d.volume}<br/>
                        Quote volume: ${d.quoteVolume}<br/>
                        Bid price: ${d.bidPrice}<br/>
                        Ask price: ${d.askPrice}<br/>
                        Trades: ${d.count}
                    `;
                }
            },

            plotOptions: {
                bubble: {
                    minSize: 8,
                    maxSize: 60,
                    dataLabels: {
                        enabled: true,
                        format: "{point.name}"
                    }
                }
            },

            series: [{
                name: "Cryptos",
                data: bubbleData
            }]
        });
    });
</script>

<h1>Crypto Bubble Chart</h1>

<div id="container"></div>

<style>
    h1 {
        text-align: center;
        font-family: Arial, sans-serif;
    }

    #container {
        width: 100%;
        height: 600px;
        margin: 0 auto;
    }
</style>