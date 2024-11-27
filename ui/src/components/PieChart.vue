<template>
  <div>
    <pie-chart :chart-data="chartData" :options="chartOptions" />
  </div>
</template>

<script>
import { Pie } from "vue-chartjs";

export default {
  name: "PieChart",
  extends: Pie,
  props: {
    chartData: {
      type: Object,
      required: true,
    },
    chartOptions: {
      type: Object,
      default: () => ({
        responsive: true,
        maintainAspectRatio: false,
        legend: {
          position: "bottom",
        },
      }),
    },
  },
  mounted() {
    // Render chart
    if (
      this.chartData &&
      this.chartData.datasets.length > 0 &&
      this.chartData.datasets[0].data.length > 0
    ) {
      this.renderChart(this.chartData, this.options);
    } else {
      console.error("No data provided for the chart.");
    }
  },
};
</script>
