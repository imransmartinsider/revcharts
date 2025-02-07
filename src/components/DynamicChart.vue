<!-- src/components/DynamicChart.vue -->
<template>
  <div class="chart-container">
    <div class="controls">
      <div class="controls-row">
        <input v-model="apiUrl" placeholder="Enter API URL" class="api-input"/>
        <select v-model="selectedChartType" class="chart-type-select">
          <option value="bar">Bar Chart</option>
          <option value="line">Line Chart</option>
          <option value="pie">Pie Chart</option>
          <option value="doughnut">Doughnut Chart</option>
          <option value="radar">Radar Chart</option>
          <option value="polarArea">Polar Area Chart</option>
        </select>
        <input v-model="chartTitle" placeholder="Chart Title" class="chart-input"/>
        <input v-model="chartColor" placeholder="Chart Color (rgba)" class="chart-input"/>
        <button @click="fetchData" class="fetch-button">Fetch</button>
        <button v-if="errorMessage" @click="retryFetch" class="retry-button">Retry</button>
        <button @click="saveConfiguration" class="save-button">Save Config</button>
        <button @click="loadConfiguration" class="load-button">Load Config</button>
        <button @click="downloadChart" class="download-button">Download</button>
      </div>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
      <div v-if="loading" class="loading-indicator">Loading...</div>
    </div>
    <canvas ref="chartCanvas" class="chart-canvas"></canvas>
  </div>
</template>

<script>
import { ref } from 'vue';
import axios from 'axios';
import { Chart, registerables } from 'chart.js';

Chart.register(...registerables);

export default {
  name: 'DynamicChart',
  setup() {
    const apiUrl = ref('');
    const selectedChartType = ref('bar'); // Default to bar chart
    const chartCanvas = ref(null);
    const errorMessage = ref('');
    const loading = ref(false); // For showing loading state
    const chartTitle = ref('');
    const chartColor = ref('rgba(75, 192, 192, 0.2)'); // Default color
    let chartInstance = null;

    const fetchData = async () => {
      if (!apiUrl.value) {
        errorMessage.value = 'Please enter an API URL for fetching';
        return;
      }

      errorMessage.value = ''; // Clear previous error messages
      loading.value = true; // Show loading indicator

      try {
        const response = await axios.get(apiUrl.value);
        const data = response.data;

        if (!Array.isArray(data) || data.length === 0) {
          errorMessage.value = 'Data is not an array or is empty';
          return;
        }

        const keys = Object.keys(data[0]);
        if (keys.length === 0) {
          errorMessage.value = 'Data does not contain any keys';
          return;
        }

        const datasets = keys.map((key) => ({
          label: key,
          data: data.map(item => item[key] || 0),
          backgroundColor: chartColor.value,
          borderColor: chartColor.value.replace('0.2', '1'),
          borderWidth: 1
        }));

        renderChart(keys, datasets);

      } catch (error) {
        console.error('Error fetching data:', error);
        errorMessage.value = 'Failed to fetch data';
      } finally {
        loading.value = false; // Hide loading indicator
      }
    };

    const renderChart = (labels, datasets) => {
      if (chartInstance) {
        chartInstance.destroy();
      }
      const ctx = chartCanvas.value.getContext('2d');
      chartInstance = new Chart(ctx, {
        type: selectedChartType.value,
        data: {
          labels: labels,
          datasets: datasets
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: {
              position: 'top',
            },
            tooltip: {
              callbacks: {
                label: function(tooltipItem) {
                  return tooltipItem.label + ': ' + tooltipItem.raw;
                }
              }
            }
          },
          scales: {
            x: {
              beginAtZero: true,
              title: {
                display: true,
                text: 'X-Axis Label'
              }
            },
            y: {
              beginAtZero: true,
              title: {
                display: true,
                text: 'Y-Axis Label'
              }
            }
          }
        }
      });
    };

    const retryFetch = () => {
      fetchData();
    };

    const saveConfiguration = () => {
      const config = {
        apiUrl: apiUrl.value,
        selectedChartType: selectedChartType.value,
        chartTitle: chartTitle.value,
        chartColor: chartColor.value
      };
      localStorage.setItem('chartConfig', JSON.stringify(config));
      alert('Configuration saved');
    };

    const loadConfiguration = () => {
      const savedConfig = localStorage.getItem('chartConfig');
      if (savedConfig) {
        const config = JSON.parse(savedConfig);
        apiUrl.value = config.apiUrl || '';
        selectedChartType.value = config.selectedChartType || 'bar';
        chartTitle.value = config.chartTitle || '';
        chartColor.value = config.chartColor || 'rgba(75, 192, 192, 0.2)';
        fetchData();
      } else {
        alert('No saved configuration found');
      }
    };

    const downloadChart = () => {
      if (chartInstance) {
        const link = document.createElement('a');
        link.href = chartInstance.toBase64Image();
        link.download = 'chart.png';
        link.click();
      } else {
        alert('No chart to download');
      }
    };

    return {
      apiUrl,
      selectedChartType,
      chartCanvas,
      errorMessage,
      loading,
      chartTitle,
      chartColor,
      fetchData,
      retryFetch,
      saveConfiguration,
      loadConfiguration,
      downloadChart
    };
  }
};
</script>

<style>
/* Import CSS file directly */
@import './styles.css';
/* Additional styles here if needed */
</style>
