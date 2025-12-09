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

const API_BASE = import.meta.env.VITE_API_BASE

async function fetchRank() {
  if (!domain.value) return

  try {
    const response = await fetch(`${API_BASE}/ranking/${domain.value}`)
    if (!response.ok) throw new Error('Failed to fetch data')

    const data = await response.json()

    latestRank.value = data.ranks[data.ranks.length - 1]
    renderChart(data.labels, data.ranks)
  } catch (err) {
    console.error('Error fetching rank:', err)
    alert('Failed to fetch rank data. Try another domain.')
  }
}

function renderChart(labels, ranks) {
  if (chartInstance) chartInstance.destroy()

  chartInstance = new Chart(chartRef.value, {
    type: 'line',
    data: {
      labels,
      datasets: [
        {
          label: domain.value,
          data: ranks,
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
