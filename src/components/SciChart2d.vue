<template>
  <div ref="chartContainer" style="width: 800px; height: 500px; border: 1px solid black;"></div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { SciChartSurface } from "scichart/Charting/Visuals/SciChartSurface";
import { NumericAxis } from "scichart/Charting/Visuals/Axis/NumericAxis";
import { XyDataSeries } from "scichart/Charting/Model/XyDataSeries";
import { FastLineRenderableSeries } from "scichart/Charting/Visuals/RenderableSeries/FastLineRenderableSeries";

export default {
  setup() {
    const chartContainer = ref(null);

    onMounted(() => {
      initializeChart(chartContainer.value);
    });

    async function fetchData() {
      const response = await fetch('https://rest.statica.pl/rest/stockquotes/gpw:PLKGHM000017?type=trades&dt_from=2010-01-01&limit=10000', {
        headers: {
          'Authorization': 'Basic ' + btoa('frontend2024:test')
        }
      });
      const data = await response.json();
      return data;
    }

    async function initializeChart(container) {
      try {
        const { sciChartSurface, wasmContext } = await SciChartSurface.create(container);
        const yAxis = new NumericAxis(wasmContext);

        const rawData = await fetchData();
        const dtValues = rawData.map(entry => entry.dt);

        const xAxis = new NumericAxis(wasmContext);

        console.log(dtValues)

        sciChartSurface.xAxes.add(xAxis);
        sciChartSurface.yAxes.add(yAxis);

        const xValues = rawData.map(entry => parseFloat(entry.price));
        const yValues = rawData.map(entry => parseFloat(entry.amount));

        const dataSeries = new XyDataSeries(wasmContext, { xValues, yValues });

        sciChartSurface.renderableSeries.add(new FastLineRenderableSeries(wasmContext, {
          dataSeries,
          stroke: "blue"
        }));

        sciChartSurface.zoomExtents();
      } catch (error) {
        console.error("Wystąpił błąd podczas inicjalizacji wykresu:", error);
      }
    }

    return {
      chartContainer
    };
  }
};
</script>

<style>
</style>
