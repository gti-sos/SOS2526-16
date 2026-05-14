<script>
    import { onMount } from "svelte";

    onMount(async () => {
        // 1. Llamada GET a la API del compañero
        const conflictData = await fetch(
            "https://sos2526-13.onrender.com/api/v2/conflict-stats"
        ).then(r => r.json());

        // 2. Llamada GET a mi api
        const evData = await fetch(
            "/api/v1/global-ev-charging-infrastructures/"
        ).then(r => r.json());

        // Extraer todos los años únicos de ambas APIs y ordenarlos
        const years = [
            ...new Set([
                ...conflictData.map(d => Number(d.year)),
                ...evData.map(d => Number(d.year))
            ])
        ].sort((a, b) => a - b);

        // Extraer ubicaciones únicas de conflictos y países únicos de EV para crear series independientes
        const locations = [...new Set(conflictData.map(d => d.location))];
        const countries = [...new Set(evData.map(d => d.country))];

        // SERIES DEL COMPAÑERO (Asignadas al Eje Y izquierdo: yAxis: 0)
        const conflictSeries = locations.map(location => ({
            name: `Conflict - ${location}`,
            type: "area",
            yAxis: 0, // Asignar al primer eje Y
            fillOpacity: 0.3, // Transparencia para que se vean los cruces de áreas
            data: years.map(year => {
                const rows = conflictData.filter(
                    d => d.location === location && Number(d.year) === year
                );

                return {
                    y: rows.length,
                    custom: {
                        source: "conflict",
                        rows
                    }
                };
            })
        }));

        // SERIES DE MI API (Asignadas al Eje Y derecho: yAxis: 1)
        const evSeries = countries.map(country => ({
            name: `EV - ${country}`,
            type: "area",
            yAxis: 1, // Asignar al segundo eje Y
            fillOpacity: 0.3, 
            data: years.map(year => {
                const row = evData.find(
                    d => d.country === country && Number(d.year) === year
                );

                return {
                    y: row ? Number(row.charging_point) : null,
                    custom: {
                        source: "ev",
                        row
                    }
                };
            })
        }));

        // Renderizar el gráfico
        Highcharts.chart("container", {
            chart: {
                type: "area"
            },
            title: {
                text: "Conflict Stats + EV Infrastructure"
            },
            subtitle: {
                text: "Datos mostrados de forma independiente (Eje Izquierdo: Conflictos | Eje Derecho: Puntos EV)"
            },
            xAxis: {
                categories: years,
                title: {
                    text: "Year"
                }
            },
            // CONFIGURACIÓN DE DOBLE EJE Y
            yAxis: [
                { // yAxis: 0 (Izquierda)
                    title: {
                        text: "Nº de Conflictos"
                    },
                    min: 0
                },
                { // yAxis: 1 (Derecha)
                    title: {
                        text: "Puntos de Carga (Charging Points)"
                    },
                    opposite: true, // Esto pone el eje a la derecha
                    min: 0
                }
            ],
            tooltip: {
                useHTML: true,
                shared: false, // Mejor false para que no mezcle ambos tooltips si pasas por el medio
                formatter: function () {
                    const custom = this.point.custom;

                    // DATOS CONFLICTOS
                    if (custom && custom.source === "conflict") {
                        if (custom.rows.length === 0) {
                            return `
                                <b>${this.series.name}</b><br/>
                                Año: ${this.x}<br/>
                                Sin conflictos
                            `;
                        }

                        return `
                            <b>${this.series.name}</b><br/>
                            Año: ${this.x}<br/>
                            Nº conflictos: ${this.y}<br/><br/>
                            ${custom.rows.map(r => `
                                Intensidad: ${r.intensity_level}<br/>
                                Tipo conflicto: ${r.conflict_type}<br/>
                                Precisión inicio: ${r.start_precision}<br/>
                            `).join("<br/>")}
                        `;
                    }

                    // DATOS EV
                    if (custom && custom.source === "ev") {
                        if (!custom.row) {
                            return `
                                <b>${this.series.name}</b><br/>
                                Año: ${this.x}<br/>
                                Sin datos EV
                            `;
                        }

                        const d = custom.row;
                        return `
                            <b>${this.series.name}</b><br/>
                            Año: ${d.year}<br/><br/>
                            Charging points: ${d.charging_point}<br/>
                            AC slow: ${d.ac_slow}<br/>
                            DC fast: ${d.dc_fast}<br/>
                            Total power kW: ${d.total_power_kw}
                        `;
                    }

                    return "No data";
                }
            },
            plotOptions: {
                area: {
                    marker: {
                        enabled: true
                    }
                }
            },
            legend: {
                enabled: true
            },
            series: [
                ...conflictSeries,
                ...evSeries
            ]
        });
    });
</script>

<svelte:head>
    <script src="https://code.highcharts.com/highcharts.js"></script>
</svelte:head>

<h1>Conflict Stats + EV Infrastructure</h1>

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