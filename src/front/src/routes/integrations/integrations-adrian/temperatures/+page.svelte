<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let temperature = 0;
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
            
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/d3/5.16.0/d3.min.js");
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.js");
            
          
            const link = document.createElement('link');
            link.rel = 'stylesheet';
            link.href = 'https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.css';
            document.head.appendChild(link);

            const c3 = window.c3;

            
            const url = 'https://api.open-meteo.com/v1/forecast?latitude=37.38&longitude=-5.98&current_weather=true';
            const res = await fetch(url);
            const json = await res.json();
            
            
            temperature = json.current_weather.temperature;

           
            c3.generate({
                bindto: '#weather-gauge',
                data: {
                    columns: [
                        ['Temperatura', temperature]
                    ],
                    type: 'gauge' 
                },
                gauge: {
                    label: {
                        format: (value) => value + "°C",
                        show: true
                    },
                    min: -10,
                    max: 50,
                    units: ' °C'
                },
                color: {
                    pattern: ['#3498db', '#f1c40f', '#e67e22', '#e74c3c'], // De frío a calor
                    threshold: {
                        values: [10, 25, 35, 45]
                    }
                },
                size: {
                    height: 250
                }
            });
            
            isLoading = false;
        } catch (error) {
            console.error("Error en la integración climática:", error);
        }
    });
</script>

<div style="text-align: center; padding: 20px; font-family: sans-serif;">
    <h1>Clima Local Actual</h1>
    
    <div id="weather-gauge"></div>

    <div style="margin-top: 30px; border-top: 1px solid #eee; padding-top: 15px;">
        
        <p><b>Ubicación:</b> Sevilla, España</p>
       
    </div>
</div>

<style>
    #weather-gauge {
        width: 100%;
        max-width: 500px;
        margin: 0 auto;
    }
</style>