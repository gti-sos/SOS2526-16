<script>
    import { onMount } from "svelte";

    // Función para cargar scripts de forma dinámica
    function loadScript(src) {
        return new Promise((resolve, reject) => {
            if (document.querySelector(`script[src="${src}"]`)) return resolve();

            const script = document.createElement("script");
            script.src = src;
            script.onload = resolve;
            script.onerror = reject;
            document.head.appendChild(script);
        });
    }

    //
    function loadCss(href) {
        if (document.querySelector(`link[href="${href}"]`)) return;

        const link = document.createElement("link");
        link.rel = "stylesheet";
        link.href = href;
        document.head.appendChild(link);
    }

    function parseAge(age) {
        return Number(String(age).replace(/\D/g, ""));
    }

    function parseBounty(bounty) {
        return Number(String(bounty).replace(/\./g, "").replace(/\D/g, ""));
    }

    onMount(async () => {
        loadCss("https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.css");

        await loadScript("https://cdnjs.cloudflare.com/ajax/libs/d3/5.16.0/d3.min.js");
        await loadScript("https://cdnjs.cloudflare.com/ajax/libs/c3/0.7.20/c3.min.js");

        const res = await fetch("https://api.api-onepiece.com/v2/characters/en");
        const characters = await res.json();

        // Filtrar personajes con edad y recompensa válida, y preparar los datos para el gráfico
        const validCharacters = characters
            .map(c => ({
                name: c.name,
                age: parseAge(c.age),
                bounty: parseBounty(c.bounty),
                crew: c.crew?.name ?? "Unknown",
                job: c.job ?? "Unknown"
            }))
            .filter(c => c.age > 0 && c.bounty > 0)
            .sort((a, b) => a.bounty - b.bounty);

        //Preparar las series para el gráfico, asignando el bounty al eje X y la edad al eje Y
        const bountyX = ["bounty_x", ...validCharacters.map(c => c.bounty)];
        const agesY = ["characters", ...validCharacters.map(c => c.age)];

        c3.generate({
            bindto: "#container",

            data: {
                xs: {
                    characters: "bounty_x"
                },
                columns: [
                    bountyX,
                    agesY
                ],
                type: "scatter"
            },

            axis: {
                x: {
                    label: "Bounty",
                    tick: {
                        fit: false,
                        format: x => x.toLocaleString()
                    }
                },
                y: {
                    label: "Age",
                    min: 0,
                    max: 100,
                    padding: {
                        top: 5,
                        bottom: 5
                    },
                    tick: {
                        values: [0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
                    }
                }
            },

            point: {
                r: 5
            },

            tooltip: {
                contents: function (d) {
                    const i = d[0].index;
                    const c = validCharacters[i];

                    if (!c) return "";

                    return `
                        <div class="custom-tooltip">
                            <b>${c.name}</b><br/>
                            Age: ${c.age}<br/>
                            Bounty: ${c.bounty.toLocaleString()}<br/>
                            Crew: ${c.crew}<br/>
                            Job: ${c.job}
                        </div>
                    `;
                }
            }
        });
    });
</script>

<h1>One Piece Characters: Bounty vs Age</h1>

<div id="container"></div>

<style>
    h1 {
        text-align: center;
        font-family: Arial, sans-serif;
    }

    #container {
        width: 100%;
        height: 650px;
        margin: 0 auto;
    }

    :global(.custom-tooltip) {
        background: white;
        border: 1px solid #ccc;
        padding: 8px;
        font-size: 13px;
        box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }
</style>