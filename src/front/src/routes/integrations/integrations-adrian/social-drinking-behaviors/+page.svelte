<script>
    import { onMount } from "svelte";

    let Highcharts;
    let isLoading = true;

   
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
            
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            Highcharts = window.Highcharts;

            
        
            const res = await fetch("https://sos2526-25.onrender.com/api/v2/social-drinking-behaviors");
            const data = await res.json();

            
            const categories = data.map(d => `${d.country} (${d.year})`);

            
            const beerSeries = data.map(d => Number(d.beer_share));
            const wineSeries = data.map(d => Number(d.wine_share));
            const spiritSeries = data.map(d => Number(d.spirit_share));

            
            Highcharts.chart("container-alcohol", {
                chart: {
                    type: "bar"
                },
                title: {
                    text: "Consumo de Alcohol por Tipo de Bebida"
                },
                xAxis: {
                    categories: categories,
                    title: { text: "País (Año)" }
                },
                yAxis: {
                    min: 0,
                    title: { text: "Litros totales consumidos" }
                },
                legend: {
                    reversed: true
                },
                plotOptions: {
                    series: {
                        stacking: "normal",
                        dataLabels: {
                            enabled: true
                        }
                    }
                },
                series: [
                    { name: "Cerveza", data: beerSeries, color: "#f1c40f" },
                    { name: "Vino", data: wineSeries, color: "#e74c3c" },
                    { name: "Espirituosos", data: spiritSeries, color: "#95a5a6" }
                ]
            });
            isLoading = false;
        } catch (error) {
            console.error("Error cargando la integración de alcohol:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px;">
    <h1>Consumo de Alcohol</h1>

    <div id="container-alcohol" style="width: 100%; height: 500px; margin: 0 auto;"></div>
</div>