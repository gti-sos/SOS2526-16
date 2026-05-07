<script>
  import { onMount } from "svelte";

  let chartContainer;
  let error = "";

  const API_URL = "https://api.restful-api.dev/objects";

  function getBrand(name) {
    const lower = name.toLowerCase();
    if (lower.includes("apple")) return "Apple";
    if (lower.includes("google")) return "Google";
    if (lower.includes("samsung")) return "Samsung";
    return "Other";
  }

  function getCapacity(item) {
    if (item.data?.capacity) {
      const match = String(item.data.capacity).match(/\d+/);
      return match ? Number(match[0]) : 0;
    }

    if (item.data?.["capacity GB"]) {
      return Number(item.data["capacity GB"]);
    }

    const match = item.name.match(/(\d+)\s?GB/i);
    return match ? Number(match[1]) : 0;
  }

  function buildSeries(devices) {
    const grouped = {};

    devices.forEach((item) => {
      const capacity = getCapacity(item);
      if (!capacity) return;

      const brand = getBrand(item.name);

      if (!grouped[brand]) grouped[brand] = [];

      grouped[brand].push({
        name: item.name,
        value: capacity
      });
    });

    return Object.entries(grouped).map(([name, data]) => ({
      name,
      data
    }));
  }

  onMount(async () => {
    try {
      const Highcharts = (await import("highcharts")).default;
      const HighchartsMore = await import("highcharts/highcharts-more");

      if (typeof HighchartsMore.default === "function") {
        HighchartsMore.default(Highcharts);
      }

      const res = await fetch(API_URL);
      const devices = await res.json();

      Highcharts.chart(chartContainer, {
        chart: {
          type: "packedbubble"
        },
        title: {
          text: "Capacidad de almacenamiento por dispositivo"
        },
        subtitle: {
          text: "API externa: restful-api.dev"
        },
        tooltip: {
          useHTML: true,
          pointFormat: "<b>{point.name}</b><br>Capacidad: {point.value} GB"
        },
        plotOptions: {
          packedbubble: {
            minSize: "20%",
            maxSize: "100%",
            zMin: 0,
            zMax: 1024,
            layoutAlgorithm: {
              splitSeries: true,
              gravitationalConstant: 0.05
            },
            dataLabels: {
              enabled: true,
              format: "{point.name}",
              style: {
                textOutline: "none"
              }
            }
          }
        },
        series: buildSeries(devices)
      });
    } catch (e) {
      error = e.message;
    }
  });
</script>

<h1>Integración con API externa</h1>

<p>
  Esta gráfica muestra dispositivos obtenidos de una API externa. El tamaño de cada burbuja representa la capacidad en GB.
</p>

{#if error}
  <p style="color: red;">{error}</p>
{/if}

<div bind:this={chartContainer} style="width: 100%; height: 600px;"></div>