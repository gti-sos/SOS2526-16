<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let isLoading = true;

    // Función para cargar Highcharts de forma segura (evita errores de servidor)
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
            // 1. CARGAR LIBRERÍAS DE MAPAS
            await loadScript("https://code.highcharts.com/maps/highmaps.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            const Highcharts = window.Highcharts;

            // 2. CARGAR EL MAPA BASE DEL MUNDO
            const worldTopology = await fetch(
                'https://code.highcharts.com/mapdata/custom/world.topo.json'
            ).then(res => res.json());

            // 3. PETICIÓN FETCH A TU PROXY (Soluciona el error 410 y cumple requisito de Proxy)
            const res = await fetch('/api/v1/proxy/map-data');
            const json = await res.json();
            
            // 4. PROCESAMIENTO DE DATOS (Mapeo de coordenadas GeoJSON)
            // Tomamos los 50 eventos más recientes
            const earthquakePoints = json.features.slice(0, 50).map(f => ({
                name: f.properties.place,
                lat: f.geometry.coordinates[1], // Latitud
                lon: f.geometry.coordinates[0], // Longitud
                z: f.properties.mag,            // Magnitud para el tamaño
                colorValue: f.properties.mag    // Magnitud para el color
            }));

            // 5. GENERACIÓN DEL MAPA
            Highcharts.mapChart('map-container', {
                chart: { 
                    map: worldTopology,
                    backgroundColor: '#f0f8ff' 
                },
                title: { text: 'Actividad Sísmica Global (Últimas 24h)' },
                subtitle: { text: 'Integración externa vía Proxy Propio | Biblioteca: Highcharts' },
                
                mapNavigation: {
                    enabled: true,
                    buttonOptions: { verticalAlign: 'bottom' }
                },

                // Gradiente de color: Amarillo (bajo) -> Rojo (alto)
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
                    type: 'mappoint', // REQUISITO: Tipo mapa de puntos (no line)
                    name: 'Terremotos',
                    data: earthquakePoints,
                    maxSize: '15%',
                    colorKey: 'colorValue', // Vincula el color a la magnitud
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
    <h1>Integración 3: Mapa de Eventos Globales</h1>
    <p>Esta visualización cumple los requisitos de <b>Proxy</b>, <b>Fetch</b> y tipo <b>Mappoint</b>.</p>
    
    {#if isLoading}
        <div style="padding: 50px;">
            <p>Conectando con la red sísmica global...</p>
            <progress></progress>
        </div>
    {/if}

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