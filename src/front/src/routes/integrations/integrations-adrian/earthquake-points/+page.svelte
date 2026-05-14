<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let isLoading = true;

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
            // Cargar librerías
            await loadScript("https://code.highcharts.com/maps/highmaps.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            const Highcharts = window.Highcharts;

            // Cargar mapa mundo genérico
            const worldTopology = await fetch(
                'https://code.highcharts.com/mapdata/custom/world.topo.json'
            ).then(res => res.json());

            // Petición fetch
            const res = await fetch('/api/v1/proxy/map-data');
            const json = await res.json();
            
            // Carga de datos
            // Tomamos los 50 eventos más recientes
            const earthquakePoints = json.features.slice(0, 50).map(f => ({
                name: f.properties.place,
                lat: f.geometry.coordinates[1], // Latitud
                lon: f.geometry.coordinates[0], // Longitud
                z: f.properties.mag,            // Magnitud para el tamaño
                colorValue: f.properties.mag    // Magnitud para el color
            }));

            // Visualización mapa
            Highcharts.mapChart('map-container', {
                chart: { 
                    map: worldTopology,
                    backgroundColor: '#f0f8ff' 
                },
                title: { text: 'Actividad Sísmica Global (Últimas 24h)' },
                
                
                mapNavigation: {
                    enabled: true,
                    buttonOptions: { verticalAlign: 'bottom' }
                },

                
                colorAxis: {
                    min: 0,
                    max: 8,
                    stops: [
                        [0, '#ffff00'],
                        [0.5, '#ff8c00'],
                        [1, '#ff0000']
                    ],
                    title: { text: 'Magnitud' }
                },

                tooltip: {
                    headerFormat: '',
                    pointFormat: '<b>Ubicación:</b> {point.name}<br>' +
                                 '<b>Coordenadas:</b> {point.lat}, {point.lon}<br>' +
                                 '<b>Magnitud:</b> {point.z} Richter'
                },

                series: [{
                    name: 'Países',
                    borderColor: '#707070',
                    nullColor: 'rgba(200, 200, 200, 0.3)',
                    showInLegend: false
                }, {
                    type: 'mappoint', 
                    name: 'Terremotos',
                    data: earthquakePoints,
                    maxSize: '15%',
                    colorKey: 'colorValue', 
                    dataLabels: {
                        enabled: false
                    }
                }]
            });

            isLoading = false;
        } catch (error) {
            console.error("Error cargando la integración del mapa:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>Mapa de Terremotos Global</h1>

    <div id="map-container"></div>
</div>

<style>
    #map-container {
        height: 600px;
        width: 100%;
        max-width: 1100px;
        margin: 0 auto;
        border: 2px solid #333;
        border-radius: 12px;
        box-shadow: 0 10px 25px rgba(0,0,0,0.2);
    }
</style>