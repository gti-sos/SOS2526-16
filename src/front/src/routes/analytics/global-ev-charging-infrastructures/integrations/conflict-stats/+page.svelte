<script>
    import { onMount } from "svelte";

    onMount(async () => {
        const response = await fetch(
            "https://sos2526-13-production.up.railway.app/api/v2/conflict-stats"
        );
        const data = await response.json();

        const years = [...new Set(data.map(d => d.year))].sort((a, b) => a - b);
        const locations = [...new Set(data.map(d => d.location))];

        const series = locations.map(location => ({
            name: location,
            data: years.map(year => {
                const conflicts = data.filter(
                    d => d.location === location && d.year === year
                );

                if (conflicts.length === 0) {
                    return {
                        y: 0,
                        conflicts: []
                    };
                }

                const avgIntensity =
                    conflicts.reduce((sum, c) => sum + Number(c.intensity_level), 0) /
                    conflicts.length;

                const conflictTypes = [
                    ...new Set(conflicts.map(c => c.conflict_type))
                ].join(", ");

                const startPrecisions = [
                    ...new Set(conflicts.map(c => c.start_precision))
                ].join(", ");

                return {
                    y: conflicts.length,
                    conflicts,
                    avgIntensity: avgIntensity.toFixed(2),
                    conflictTypes,
                    startPrecisions
                };
            })
        }));

        Highcharts.chart("container", {
            chart: {
                type: "area"
            },

            title: {
                text: "Distribución porcentual de conflictos por localización"
            },

            subtitle: {
                text: "Datos obtenidos de la API conflict-stats"
            },

            xAxis: {
                categories: years,
                title: {
                    text: "Año"
                }
            },

            yAxis: {
                labels: {
                    format: "{value}%"
                },
                title: {
                    enabled: false
                }
            },

            tooltip: {
                formatter: function () {
                    if (!this.point || this.point.y === 0) {
                        return `
                            <b>${this.series.name}</b><br/>
                            Año: ${this.x}<br/>
                            No hay conflictos registrados
                        `;
                    }

                    return `
                        <b>${this.series.name}</b><br/>
                        Año: ${this.x}<br/>
                        Nº de conflictos: ${this.point.y}<br/>
                        Porcentaje: ${this.percentage.toFixed(1)}%<br/>
                        Intensidad media: ${this.point.avgIntensity}<br/>
                        Tipos de conflicto: ${this.point.conflictTypes}<br/>
                        Precisión de inicio: ${this.point.startPrecisions}
                    `;
                }
            },

            plotOptions: {
                area: {
                    stacking: "percent",
                    marker: {
                        enabled: false
                    }
                }
            },

            series
        });
    });
</script>

<svelte:head>
    <script src="https://code.highcharts.com/highcharts.js"></script>
</svelte:head>

<h1>Conflict Stats</h1>

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