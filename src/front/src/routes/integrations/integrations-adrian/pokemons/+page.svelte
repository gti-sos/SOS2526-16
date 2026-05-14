<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let pokemonData = [];
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
            // Cargar módulos de la librería
            await loadScript("https://code.highcharts.com/highcharts.js");
            await loadScript("https://code.highcharts.com/modules/variable-pie.js");
            await loadScript("https://code.highcharts.com/modules/accessibility.js");
            const Highcharts = window.Highcharts;

            // 2. Petición fetch. Obtenemos 5 Pokemons
            const ids = [1, 4, 7, 25, 143]; // Bulbasaur, Charmander, Squirtle, Pikachu, Snorlax
            const responses = await Promise.all(ids.map(id => fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)));
            const data = await Promise.all(responses.map(r => r.json()));
            
            pokemonData = data;

            // Carga del gráfico
            // Cada punto necesita: y (valor para el ángulo) y z (valor para el radio)
            const chartData = data.map(p => ({
                name: p.name.toUpperCase(),
                y: p.base_experience, // Determina el ángulo
                z: p.weight           // Determina el radio (lo larga que es la porción)
            }));

            // Visualización del gráfico
            Highcharts.chart('container-variable-pie', {
                chart: {
                    type: 'variablepie', 
                    backgroundColor: '#fdfdfd'
                },
                title: { text: 'Comparativa de Experiencia y Peso Pokémon' },
                subtitle: { text: 'Ángulo: Experiencia Base | Radio: Peso del Pokémon' },
                tooltip: {
                    headerFormat: '',
                    pointFormat: '<span style="color:{point.color}">\u25CF</span> <b> {point.name}</b><br/>' +
                        'Experiencia Base (Ángulo): <b>{point.y}</b><br/>' +
                        'Peso (Radio): <b>{point.z}</b><br/>'
                },
                series: [{
                    minPointSize: 10,
                    innerSize: '20%',
                    zMin: 0,
                    name: 'pokemons',
                    data: chartData
                }]
            });

            isLoading = false;
        } catch (error) {
            console.error("Error en la integración individual:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>Pokemons</h1>
    </div>
   

    <div id="container-variable-pie"></div>
