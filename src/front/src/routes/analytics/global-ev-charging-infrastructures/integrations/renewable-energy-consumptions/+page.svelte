<script>
    import { onMount } from "svelte";

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
        await loadScript("https://code.highcharts.com/highcharts.js");
        await loadScript("https://code.highcharts.com/modules/heatmap.js");

        const res = await fetch("https://api-sos.pablogamero.com/api/v1/renewable-energy-consumptions?limit=100");
        const data = await res.json();

        const countries = [...new Set(data.map(d => d.country))];
        const years = [...new Set(data.map(d => d.year))].sort((a, b) => a - b);

        const heatmapData = data.map(d => {
            const x = years.indexOf(d.year);
            const y = countries.indexOf(d.country);

            const total =
                Number(d.wind) +
                Number(d.hydro) +
                Number(d.solar) +
                Number(d.other);

            return [x, y, total];
        });

        Highcharts.chart("container", {
            chart: {
                type: "heatmap"
            },

            title: {
                text: "Renewable Energy Consumption (All Countries & Years)"
            },

            xAxis: {
                categories: years,
                title: { text: "Year" }
            },

            yAxis: {
                categories: countries,
                title: { text: "Country" },
                reversed: true
            },

            colorAxis: {
                min: 0,
                minColor: "#FFFFFF",
                maxColor: "#007bff"
            },

            tooltip: {
                formatter: function () {
                    return `
                        <b>${countries[this.point.y]}</b><br/>
                        Año: ${years[this.point.x]}<br/>
                        Consumo total: ${this.point.value}
                    `;
                }
            },

            series: [{
                name: "Energy",
                borderWidth: 0,
                data: heatmapData
            }]
        });
    });
</script>

<h1>Heatmap Renewable Energy</h1>

<div id="container"></div>

<style>
    h1 {
        text-align: center;
        font-family: Arial, sans-serif;
    }

    #container {
        width: 100%;
        height: 700px;
        margin: 0 auto;
    }
</style>