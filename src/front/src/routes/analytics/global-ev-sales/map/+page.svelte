<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let data = [];
    let years = [];
    let selectedYear = null;
    let mapContainer;
    let mapInstance;
    let markerLayerGroup;

    // Diccionario de coordenadas (Latitud, Longitud) para colocar los marcadores
    // Asegúrate de que las claves están en minúsculas para que coincidan con tu API
    const countryCoords = {
        'spain': [40.4637, -3.7492],
        'germany': [51.1657, 10.4515],
        'canada': [56.1304, -106.3468],
        'finland': [61.9241, 25.7482],
        'france': [46.2276, 2.2137],
        'norway': [60.472, 8.4689],
        'usa': [37.0902, -95.7129],
        'china': [35.8617, 104.1954],
        'uk': [55.3781, -3.4360],
        'italy': [41.8719, 12.5674],
        'sweden': [60.1282, 18.6435],
        'netherlands': [52.1326, 5.2913],
        'brazil': [-14.2350, -51.9253],
        'mexico': [23.6345, -102.5528]
    };

    // Función para cargar los scripts y estilos de Leaflet dinámicamente
    async function initLeaflet() {
        return new Promise((resolve, reject) => {
            if (window.L) return resolve(window.L);

            const link = document.createElement('link');
            link.rel = 'stylesheet';
            link.href = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.css';
            document.head.appendChild(link);

            const script = document.createElement('script');
            script.src = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.js';
            script.onload = () => resolve(window.L);
            script.onerror = reject;
            document.head.appendChild(script);
        });
    }

    async function renderMap() {
        if (!mapInstance || !selectedYear || !window.L) return;

        // Limpiamos los marcadores del año anterior
        markerLayerGroup.clearLayers();

        // Filtramos los datos por el año seleccionado
        const filteredData = data.filter(d => Number(d.year) === Number(selectedYear));

        filteredData.forEach(d => {
            const countryKey = d.region ? d.region.toLowerCase() : '';
            const coords = countryCoords[countryKey];

            if (coords) {
                // Calculamos el radio de la burbuja (usamos raíz cuadrada para que no sean gigantes)
                const radiusSize = Math.max(5, Math.sqrt(Number(d.value) || 0) / 10);

                // Creamos un marcador circular geolocalizado
                const circleMarker = window.L.circleMarker(coords, {
                    radius: radiusSize,
                    fillColor: "#ff7800",
                    color: "#000",
                    weight: 1,
                    opacity: 1,
                    fillOpacity: 0.7
                });

                // Creamos el popup (la tarjeta que se abre al hacer clic)
                const popupContent = `
                    <div style="font-family: sans-serif; min-width: 150px;">
                        <h3 style="margin: 0 0 5px 0; border-bottom: 1px solid #ccc; padding-bottom: 5px; color: #333;">${d.region}</h3>
                        <p style="margin: 2px 0;"><b>Año:</b> ${d.year}</p>
                        <p style="margin: 2px 0;"><b>Ventas (Valor):</b> ${d.value}</p>
                        <p style="margin: 2px 0;"><b>Impacto Econ.:</b> ${d.economic_impact}</p>
                        <p style="margin: 2px 0;"><b>Categoría:</b> ${d.category}</p>
                        <p style="margin: 2px 0;"><b>Motor:</b> ${d.powertrain}</p>
                    </div>
                `;

                circleMarker.bindPopup(popupContent);
                circleMarker.addTo(markerLayerGroup);
            }
        });
    }

    function handleYearChange() {
        renderMap();
    }

    onMount(async () => {
        if (!browser) return;

        try {
            // 1. Cargar Leaflet
            const L = await initLeaflet();

            // 2. Inicializar el mapa vacío centrado en el Atlántico
            mapInstance = L.map(mapContainer).setView([20, 0], 2);
            
            // Añadir la capa de mapa base (OpenStreetMap)
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 19,
                attribution: '© OpenStreetMap contributors'
            }).addTo(mapInstance);

            // Crear una capa para agrupar nuestros marcadores y poder borrarlos al cambiar de año
            markerLayerGroup = L.layerGroup().addTo(mapInstance);

            // 3. Cargar nuestros datos de la API
            await fetch('/api/v1/global-ev-sales/loadInitialData');
            const res = await fetch('/api/v1/global-ev-sales');
            
            if (res.ok) {
                data = await res.json();
                
                // Extraer años únicos para el desplegable
                years = [...new Set(data.map(d => Number(d.year)))]
                    .filter(y => !isNaN(y))
                    .sort((a, b) => a - b);

                if (years.length > 0) {
                    selectedYear = years[0]; // Seleccionamos el primer año por defecto
                    renderMap(); // Dibujamos los marcadores
                }
            }
        } catch (error) {
            console.error("Error al cargar el mapa:", error);
        }
    });
</script>

<svelte:head>
    <title>Geospatial - Global EV Sales</title>
</svelte:head>

<main>
    <h2>🌍 Mapa Interactivo de Ventas de Vehículos Eléctricos</h2>

    <div class="controls">
        <label for="year-select">Filtrar por Año:</label>
        <select id="year-select" bind:value={selectedYear} onchange={handleYearChange}>
            {#if years.length === 0}
                <option disabled>Cargando datos...</option>
            {:else}
                {#each years as y}
                    <option value={y}>{y}</option>
                {/each}
            {/if}
        </select>
    </div>

    <div class="map-wrapper">
        <div bind:this={mapContainer} style="width: 100%; height: 100%;"></div>
    </div>

    <div class="actions">
        <a href="/global-ev-sales" class="btn-back">← Volver a la tabla</a>
        <a href="/analytics/global-ev-sales" class="btn-analytics">Ir a la Gráfica Principal</a>
    </div>
</main>

<style>
    main {
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
        font-family: Arial, sans-serif;
    }
    
    h2 {
        text-align: center;
        color: #2c3e50;
        margin-bottom: 20px;
    }

    .controls {
        text-align: center;
        margin-bottom: 20px;
        background-color: #f8f9fa;
        padding: 15px;
        border-radius: 8px;
        border: 1px solid #dee2e6;
    }

    .controls label {
        font-weight: bold;
        margin-right: 10px;
        color: #495057;
    }

    .controls select {
        padding: 8px 15px;
        border-radius: 5px;
        border: 1px solid #ced4da;
        font-size: 16px;
        cursor: pointer;
    }

    .map-wrapper {
        height: 600px;
        width: 100%;
        border-radius: 10px;
        overflow: hidden;
        box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        border: 2px solid #e9ecef;
        margin-bottom: 20px;
    }

    .actions {
        display: flex;
        justify-content: center;
        gap: 15px;
    }

    .btn-back {
        padding: 10px 20px;
        background-color: #6c757d;
        color: white;
        text-decoration: none;
        border-radius: 5px;
        font-weight: bold;
        transition: background-color 0.3s;
    }

    .btn-analytics {
        padding: 10px 20px;
        background-color: #33b5e5;
        color: white;
        text-decoration: none;
        border-radius: 5px;
        font-weight: bold;
        transition: background-color 0.3s;
    }

    .btn-back:hover { background-color: #5a6268; }
    .btn-analytics:hover { background-color: #2aa1c0; }
</style>