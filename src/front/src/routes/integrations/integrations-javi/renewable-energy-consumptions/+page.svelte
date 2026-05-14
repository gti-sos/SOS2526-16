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

        // 1. OBTENER DATOS
        const energyRes = await fetch("https://api-sos.pablogamero.com/api/v1/renewable-energy-consumptions?limit=100");
        const energyData = await energyRes.json();

        // Llamada a mi api
        const evRes = await fetch("/api/v1/global-ev-charging-infrastructures/");
        const evData = await evRes.json();

        
        console.log("Datos de Energía:", energyData.length);
        console.log("MIS DATOS EV:", evData);

        // 2. EXTRAER AÑOS Y PAÍSES
        const years = [
            ...new Set([
                ...energyData.map(d => Number(d.year)),
                ...evData.map(d => Number(d.year))
            ])
        ].sort((a, b) => a - b);

        // Extraer países únicos para cada dataset para crear categorías en el eje Y
        const energyCountries = [...new Set(energyData.map(d => d.country))];
        const evCountries = [...new Set(evData.map(d => d.country))];

        // Combinar ambos conjuntos de países para tener categorías completas en el eje Y, diferenciando entre energía y EV
        const combinedYCategories = [
            ...energyCountries.map(c => `${c} (Energy)`),
            ...evCountries.map(c => `${c} (EV)`)
        ];

        //Data para el heatmap de energía (Azul)
        const heatmapEnergyData = [];
        energyCountries.forEach(country => {
            const yIndex = combinedYCategories.indexOf(`${country} (Energy)`);
            
            // Para cada año, buscar el consumo total de energía renovable (suma de wind, hydro, solar y other) para ese país y año
            years.forEach((year, xIndex) => {
                const found = energyData.find(d => d.country === country && Number(d.year) === year);
                let total = null;

                // Si se encuentra un registro para ese país y año, sumar las fuentes de energía renovable para obtener el consumo total
                if (found) {
                    total = Number(found.wind) + Number(found.hydro) + Number(found.solar) + Number(found.other);
                }
                // Agregar el punto al dataset del heatmap, diferenciando entre energía y EV mediante la propiedad "source"
                heatmapEnergyData.push({
                    x: xIndex,
                    y: yIndex,
                    value: total,
                    custom: found || { country, year, empty: true },
                    source: "energy"
                });
            });
        });

        //Data para el heatmap de EV (Amarillo)
        const heatmapEVData = [];
        evCountries.forEach(country => {
            const yIndex = combinedYCategories.indexOf(`${country} (EV)`);
            // Para cada año, buscar el número de puntos de carga para ese país y año
            years.forEach((year, xIndex) => {
                const found = evData.find(d => d.country === country && Number(d.year) === year);
                let totalPoints = null;

                if (found) {
                    totalPoints = Number(found.charging_point);
                }

                heatmapEVData.push({
                    x: xIndex,
                    y: yIndex,
                    value: totalPoints,
                    custom: found || { country, year, empty: true },
                    source: "ev"
                });
            });
        });

        // 5. PINTAR LA GRÁFICA
        Highcharts.chart("container", {
            chart: {
                type: "heatmap",
                backgroundColor: "#fcfcfc"
            },
            title: {
                text: "Renewable Energy (Azul) vs EV Infrastructure (Amarillo)"
            },
            xAxis: {
                categories: years,
                title: { text: "Year" }
            },
            yAxis: {
                categories: combinedYCategories,
                title: { text: "Country" },
                reversed: true,
                // Esto fuerza a que se muestren todos los nombres sin ocultarlos
                labels: {
                    step: 1
                }
            },
            colorAxis: [
                { // Escala 0: Energía (Azules)
                    min: 0,
                    minColor: "#e6f2ff",
                    maxColor: "#005ce6",
                    nullColor: '#f4f4f4' 
                },
                { // Escala 1: EV (Amarillos)
                    min: 0,
                    minColor: "#ffffe6", // Amarillo muy clarito
                    maxColor: "#ffcc00", // Amarillo fuerte/naranja
                    nullColor: '#f4f4f4'
                }
            ],
            tooltip: {
                useHTML: true,
                formatter: function () {
                    const custom = this.point.custom;

                    if (custom.empty) {
                        return `<b>${custom.country.toUpperCase()}</b><br/>Año: ${custom.year}<br/><i>Sin datos registrados</i>`;
                    }

                    if (this.point.source === "energy") {
                        return `
                            <b>${custom.country.toUpperCase()} (Energy)</b><br/>
                            Año: ${custom.year}<br/><br/>
                            Consumo Total: <b>${this.point.value.toFixed(2)}</b>
                        `;
                    }

                    if (this.point.source === "ev") {
                        return `
                            <b>${custom.country.toUpperCase()} (EV)</b><br/>
                            Año: ${custom.year}<br/><br/>
                            Charging Points: <b>${this.point.value}</b>
                        `;
                    }
                }
            },
            series: [
                {
                    name: "Energy Consumption",
                    borderWidth: 1,
                    borderColor: '#ffffff',
                    data: heatmapEnergyData,
                    colorAxis: 0
                },
                {
                    name: "EV Charging Points",
                    borderWidth: 1,
                    borderColor: '#ffffff',
                    data: heatmapEVData,
                    colorAxis: 1 // Conecta con los amarillos
                }
            ]
        });
    });
</script>

<h1>Heatmap: Energy & EV Infrastructure</h1>

<div id="container"></div>

<style>
    h1 {
        text-align: center;
        font-family: Arial, sans-serif;
        color: #333;
    }

    #container {
        width: 100%;
        /* Le damos bastante altura para que quepan todos los países */
        height: 1200px; 
        margin: 0 auto;
    }
</style>