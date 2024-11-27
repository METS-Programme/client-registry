<template>
  <v-container fluid>
    <v-card class="my-2">
      <v-card-title>
        {{ $t("menu_dashboard") }}
        <v-spacer />
      </v-card-title>
    </v-card>
    <v-spacer />

    <!-- Metrics Section -->
    <v-card class="pt-2">
      <v-row class="my-2">
        <!-- Loop through metrics array -->
        <v-col
          v-for="(metric, index) in metrics"
          :key="index"
          cols="12"
          sm="6"
          md="3"
        >
          <v-card class="mx-auto" max-width="400" outlined>
            <v-list-item three-line>
              <v-list-item-content>
                <div class="text-overline mb-4">
                  {{ metric.title }}
                </div>
                <v-list-item-title class="text-h5 mb-1">
                  {{ metric.value }}
                </v-list-item-title>
                <v-list-item-subtitle>
                  {{ metric.description }}
                </v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-avatar tile size="80" :color="metric.color">
                <v-icon>{{ metric.icon }}</v-icon>
              </v-list-item-avatar>
            </v-list-item>
          </v-card>
        </v-col>
      </v-row>
    </v-card>

    <v-spacer />

    <!-- Matches Section -->
    <v-card class="pt-2">
      <v-row>
        <!-- Pie Chart -->
        <v-col cols="12" sm="6">
          <v-card class="pa-4" outlined style="height: 590px;">
            <PieChart
              :chart-data="chartData"
              :options="chartOptions"
              style="height: 100%;"
            ></PieChart>
          </v-card>
        </v-col>

        <!-- Table -->
        <v-col cols="12" sm="6">
          <v-card class="pa-4" outlined style="height: 590px;">
            <v-data-table
              :headers="tableHeaders"
              :items="tableItems"
              class="elevation-1"
              dense
              style="height: calc(100% - 56px); overflow: auto;"
            >
              <template v-slot:top>
                <v-toolbar flat>
                  <v-toolbar-title>Match Data</v-toolbar-title>
                  <v-spacer />
                </v-toolbar>
              </template>
            </v-data-table>
          </v-card>
        </v-col>
      </v-row>
    </v-card>
  </v-container>
</template>

<script>
import { Pie } from "vue-chartjs";
import axios from "axios";

export default {
  name: "Dashboard",
  components: {
    PieChart: {
      extends: Pie,
      props: {
        chartData: {
          type: Object,
          required: true,
        },
        options: {
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
        this.renderChart(this.chartData, this.options);
      },
    },
  },
  data() {
    return {
      // Table configurations
      tableHeaders: [
        { text: "CRUID", value: "id" },
        { text: "Surname", value: "family" },
        { text: "Given Name", value: "given" },
        { text: "Source", value: "source" },
        { text: "Source ID", value: "source_id" },
      ],
      tableItems: [],
      // Metrics and Chart
      metrics: [],
      chartData: {
        labels: [
          "Total Matches",
          "Potential Matches",
          "Conflict Matches",
          "Auto Matches",
        ],
        datasets: [
          {
            label: "Match Overview", // Optional
            backgroundColor: ["#4caf50", "#f44336", "#2196f3", "#ffeb3b"],
            hoverBackgroundColor: ["#66bb6a", "#e57373", "#64b5f6", "#fff176"], // Optional
            data: [0, 0, 0, 0], 
          },
        ],
      },

      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        legend: {
          position: "bottom",
        },
        title: {
          display: true,
          text: "Matches Overview",
        },
      },
    };
  },
  created() {
    this.$store.state.progress.enable = true;
    this.$store.state.progress.width = "300px";
    this.$store.state.progress.title = "Loading data";

    // Fetch metrics and chart data
    this.fetchMatchIssues();
    this.fetchStats();
  },
  methods: {
    fetchMatchIssues() {
      axios
        .get(`/ocrux/match/get-match-issues`)
        .then((resp) => {
          this.$store.state.progress.enable = false;
          this.tableItems = resp.data;
        })
        .catch(this.handleError);
    },
    fetchStats() {
      axios
        .get("/ocrux/match/cr-stats")
        .then((resp) => {
          const stats = resp.data;
          // Populate chart data dynamically
          this.chartData.datasets[0].data = [
            stats.totalMatches,
            stats.potentialMatches,
            stats.conflictMatches,
            stats.autoMatches,
          ];

          // Update metrics dynamically
          this.metrics = [
            {
              title: "Total Facilities",
              value: stats.totalFacilities,
              description: "No. of facilities submitting records.",
              color: "blue",
              icon: "mdi-hospital",
            },
            {
              title: "Total Patients",
              value: stats.totalPatients,
              description: "Patients admitted this week.",
              color: "green",
              icon: "mdi-account",
            },
            {
              title: "Potential Matches",
              value: stats.potentialMatches,
              description: "Potential patient matches.",
              color: "orange",
              icon: "mdi-calendar-clock",
            },
            {
              title: "Conflict Matches",
              value: stats.conflictMatches,
              description: "Conflict patient matches.",
              color: "orange",
              icon: "mdi-calendar-clock",
            },
            {
              title: "Auto Matches",
              value: stats.autoMatches,
              description: "Auto patient matches.",
              color: "red",
              icon: "mdi-flask-outline",
            },
          ];
        })
        .catch(this.handleError);
    },
    handleError() {
      this.$store.state.progress.enable = false;
      this.$store.state.alert.show = true;
      this.$store.state.alert.width = "500px";
      this.$store.state.alert.msg = this.$t("something_wrong");
      this.$store.state.alert.type = "error";
    },
  },
};
</script>

<style scoped>
/* Add styling if necessary */
</style>
