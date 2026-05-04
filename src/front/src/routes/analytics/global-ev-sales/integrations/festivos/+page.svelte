<script>
    import { onMount } from 'svelte';
    let festivos = $state([]);
    let error = $state(false);

    onMount(async () => {
        try {
            const res = await fetch('https://date.nager.at/api/v3/PublicHolidays/2024/ES');
            if (res.ok) {
                festivos = await res.json();
            } else {
                error = true;
            }
        } catch (e) {
            error = true;
        }
    });
</script>

<main style="padding: 20px; font-family: sans-serif;">
    <h2>Calendario de Festivos en España (2024)</h2>
    <p>Uso de API externa textual (Nager.Date) sin widget.</p>

    {#if error}
        <p style="color: red;">Error al cargar los datos.</p>
    {:else}
        <table border="1" style="width: 100%; border-collapse: collapse; text-align: left;">
            <thead style="background-color: #f2f2f2;">
                <tr>
                    <th style="padding: 10px;">Fecha</th>
                    <th style="padding: 10px;">Nombre Local</th>
                    <th style="padding: 10px;">Nombre Inglés</th>
                </tr>
            </thead>
            <tbody>
                {#each festivos as f}
                    <tr>
                        <td style="padding: 10px;">{f.date}</td>
                        <td style="padding: 10px;">{f.localName}</td>
                        <td style="padding: 10px;">{f.name}</td>
                    </tr>
                {/each}
            </tbody>
        </table>
    {/if}
</main>