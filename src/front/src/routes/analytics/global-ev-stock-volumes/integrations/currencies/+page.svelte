<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    async function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) return resolve();
            const script = document.createElement('script');
            script.src = src; script.onload = resolve; document.head.appendChild(script);
        });
    }

    onMount(async () => {
        if (!browser) return;
        try {
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/d3/5.16.0/d3.min.js");
            await loadScript("https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.js");
            const c3 = window.c3;

            // FETCH DIRECTO (CORS ABIERTO)
            const res = await fetch('https://api.frankfurter.app/latest?to=USD,GBP,JPY,CAD,CHF');
            const json = await res.json();
            const rates = json.rates;

            // Preparamos los datos para el Donut
            const columns = Object.keys(rates).map(key => [key, rates[key]]);

            c3.generate({
                bindto: '#chart-donut-currency',
                data: {
                    columns: columns,
                    type: 'donut' // REQUISITO: No es line
                },
                donut: { title: "Valor vs EUR" }
            });
        } catch (e) { console.error(e); }
    });
</script>

<div style="text-align: center; padding: 20px;">
    <h1>Integración Externa: Divisas (C3.js Donut)</h1>
    <div id="chart-donut-currency" style="height: 400px;"></div>
</div>