<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let universidades = $state([]);
    let cargando = $state(true);
    let errorMensaje = $state('');

    onMount(async () => {
        if (!browser) return;

        try {
            // Fetch directo a la API pública de Universidades (sin CORS)
            // Filtramos por España para tener datos locales conocidos
            const res = await fetch('http://universities.hipolabs.com/search?country=Spain');
            
            if (!res.ok) throw new Error("No se pudo contactar con el registro universitario.");
            
            const data = await res.json();

            // La API devuelve muchas, así que nos quedamos con las 10 primeras
            // (Puedes cambiar este número si quieres mostrar más)
            universidades = data.slice(0, 10);
            cargando = false;

        } catch (e) {
            console.error(e);
            errorMensaje = "Error al cargar el directorio de universidades.";
            cargando = false;
        }
    });
</script>

<main style="max-width: 800px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2 style="text-align: center; color: #333;">Directorio Universitario</h2>
    <p style="text-align: center; color: #666; margin-bottom: 30px;">
        Uso textual de la API <strong>Hipolabs</strong>. Mostrando datos en tabla HTML sin widgets.
    </p>

    {#if errorMensaje}
        <div style="background-color: #ffebee; color: #c62828; padding: 15px; border-radius: 5px; text-align: center; font-weight: bold;">
            {errorMensaje}
        </div>
    {/if}

    <div style="background: white; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); padding: 20px;">
        {#if cargando}
            <div style="text-align: center; padding: 40px; color: #888; font-style: italic;">
                Recopilando datos de las facultades...
            </div>
        {:else if universidades.length > 0}
            <table style="width: 100%; border-collapse: collapse; text-align: left;">
                <thead>
                    <tr style="background-color: #f8f9fa; border-bottom: 2px solid #dee2e6;">
                        <th style="padding: 12px 15px; color: #495057;">#</th>
                        <th style="padding: 12px 15px; color: #495057;">Nombre de la Institución</th>
                        <th style="padding: 12px 15px; color: #495057;">Sitio Web Oficial</th>
                    </tr>
                </thead>
                <tbody>
                    {#each universidades as uni, i}
                        <tr style="border-bottom: 1px solid #eee; transition: background-color 0.2s;">
                            <td style="padding: 12px 15px; color: #6c757d;">{i + 1}</td>
                            <td style="padding: 12px 15px; font-weight: 500; color: #212529;">{uni.name}</td>
                            <td style="padding: 12px 15px;">
                                <a href={uni.web_pages[0]} target="_blank" rel="noopener noreferrer" style="color: #0d6efd; text-decoration: none;">
                                    {uni.web_pages[0]}
                                </a>
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        {/if}
    </div>

    <div style="margin-top: 30px; text-align: center;">
        <a href="/analytics/global-ev-sales/integrations" 
           style="display: inline-block; background-color: #333; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; font-weight: bold;">
           ← Volver al Menú Principal
        </a>
    </div>
</main>