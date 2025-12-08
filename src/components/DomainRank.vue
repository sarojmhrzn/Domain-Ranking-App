<script setup>
import { ref } from 'vue'
import {
  Chart,
  LineController,
  LineElement,
  PointElement,
  CategoryScale,
  LinearScale,
  Tooltip,
  Legend,
} from 'chart.js'

Chart.register(
  LineController,
  LineElement,
  PointElement,
  CategoryScale,
  LinearScale,
  Tooltip,
  Legend,
)

const domain = ref('onlinekhabar.com')
const latestRank = ref(null)
const chartRef = ref(null)
let chartInstance = null

// Dummy rank data (your backend will replace this later)
const dummyData = {
  labels: [
    'Oct 27',
    'Oct 28',
    'Oct 29',
    'Oct 30',
    'Oct 31',
    'Nov 1',
    'Nov 2',
    'Nov 3',
    'Nov 4',
    'Nov 5',
    'Nov 6',
    'Nov 7',
    'Nov 8',
    'Nov 9',
    'Nov 10',
    'Nov 11',
    'Nov 12',
    'Nov 13',
    'Nov 14',
    'Nov 15',
    'Nov 16',
    'Nov 17',
    'Nov 18',
    'Nov 19',
    'Nov 20',
  ],
  ranks: [
    11890, 11800, 11710, 11620, 11530, 11420, 11310, 11200, 11100, 10950, 10720, 10580, 9500, 9100,
    8600, 8000, 7990, 8010, 8005, 8012, 8020, 8030, 8050, 8080, 8100,
  ],
}

function fetchRank() {
  // Fill latest rank
  latestRank.value = dummyData.ranks[dummyData.ranks.length - 1]

  renderChart()
}

function renderChart() {
  if (chartInstance) chartInstance.destroy()

  chartInstance = new Chart(chartRef.value, {
    type: 'line',
    data: {
      labels: dummyData.labels,
      datasets: [
        {
          label: domain.value,
          data: dummyData.ranks,
          borderColor: '#3b82f6',
          backgroundColor: '#3b82f6',
          pointRadius: 4,
          tension: 0.3,
        },
      ],
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          display: true,
          position: 'top',
        },
      },
      scales: {
        y: {
          reverse: true,
          title: { display: true, text: 'Rank' },
        },
      },
    },
  })
}
</script>

<template>
  <div class="page">
    <h1 class="title">Get your domain rank</h1>
    <p class="subtitle">It uses tranco api to get ranking data under the hood.</p>

    <!-- SEARCH BAR -->
    <div class="search">
      <input v-model="domain" placeholder="onlinekhabar.com" class="input" />
      <button class="btn" @click="fetchRank">Get rank</button>
    </div>

    <!-- LATEST RANK INFO -->
    <p v-if="latestRank" class="latest">
      The latest rank for <strong>{{ domain }}</strong> is {{ latestRank }}
    </p>

    <!-- CHART -->
    <div class="chart-container">
      <canvas ref="chartRef"></canvas>
    </div>
  </div>
</template>

<style scoped>
.page {
  max-width: 900px;
  margin: auto;
  padding: 30px;
  font-family: Arial, sans-serif;
}

.title {
  font-size: 28px;
  font-weight: 700;
}

.subtitle {
  color: #555;
  margin-bottom: 20px;
}

.search {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

.btn {
  background: #3b82f6;
  color: white;
  padding: 10px 18px;
  border: none;
  border-radius: 6px;
  font-size: 15px;
  cursor: pointer;
}

.btn:hover {
  background: #2563eb;
}

.latest {
  margin-bottom: 20px;
  font-size: 16px;
}

.chart-container {
  margin-top: 20px;
}
</style>
