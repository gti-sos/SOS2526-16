<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let usuarios = $state([]);
    let cargando = $state(true);
    let errorMensaje = $state('');

    onMount(async () => {
        if (!browser) return;

        try {
            // Petición directa, 100% segura (HTTPS) y a prueba de balas
            const res = await fetch('https://randomuser.me/api/?results=10');
            
            if (!res.ok) throw new Error("No se pudo contactar con el servidor de usuarios.");
            
            const data = await res.json();
            usuarios = data.results;
            cargando = false;

        } catch (e) {
            console.error(e);
            errorMensaje = "Error al cargar el directorio de usuarios.";
            cargando = false;
        }
    });
</script>

<main style="max-width: 900px; margin: 0 auto; padding: 20px; font-family: sans-serif;">
    <h2 style="text-align: center; color: #2c3e50;">Directorio de Clientes Internacionales</h2>
    <p style="text-align: center; color: #7f8c8d; margin-bottom: 30px;">
        Uso textual de la API <strong>RandomUser</strong> (100% HTTPS). Mostrando datos en tabla HTML.
    </p>

    {#if errorMensaje}
        <div style="background-color: #fee2e2; color: #991b1b; padding: 15px; border-radius: 8px; text-align: center; font-weight: bold;">
            {errorMensaje}
        </div>
    {/if}

    <div style="background: white; border-radius: 12px; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); overflow: hidden;">
        {#if cargando}
            <div style="text-align: center; padding: 60px; color: #64748b; font-style: italic;">
                Descargando perfiles desde el servidor seguro...
            </div>
        {:else if usuarios.length > 0}
            <div style="overflow-x: auto;">
                <table style="width: 100%; border-collapse: collapse; text-align: left; min-width: 600px;">
                    <thead>
                        <tr style="background-color: #f8fafc; border-bottom: 2px solid #e2e8f0;">
                            <th style="padding: 16px; color: #475569;">Nombre Completo</th>
                            <th style="padding: 16px; color: #475569;">País de Origen</th>
                            <th style="padding: 16px; color: #475569;">Correo Electrónico</th>
                            <th style="padding: 16px; color: #475569;">Edad</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#each usuarios as user}
                            <tr style="border-bottom: 1px solid #f1f5f9; transition: background-color 0.2s;">
                                <td style="padding: 16px; font-weight: 500; color: #334155;">
                                    {user.name.first} {user.name.last}
                                </td>
                                <td style="padding: 16px; color: #64748b;">
                                    {user.location.country}
                                </td>
                                <td style="padding: 16px; color: #0284c7;">
                                    {user.email}
                                </td>
                                <td style="padding: 16px; color: #64748b;">
                                    {user.dob.age} años
                                </td>
                            </tr>
                        {/each}
                    </tbody>
                </table>
            </div>
        {/if}
    </div>

    <div style="margin-top: 40px; text-align: center;">
        <a href="/analytics/global-ev-sales/integrations" 
           style="display: inline-block; background-color: #1e293b; color: white; padding: 12px 24px; text-decoration: none; border-radius: 6px; font-weight: bold; transition: background-color 0.2s;">
           ← Volver al Menú Principal
        </a>
    </div>
</main>