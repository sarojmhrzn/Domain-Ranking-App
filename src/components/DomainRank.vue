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

const domain = ref('')
const labels = ref([])
const datasets = ref([])
const latestRanks = ref({})
const chartRef = ref(null)
let chartInstance = null
let isLoading = ref(false)
let noRankMessages = ref({})

const API_BASE = import.meta.env.VITE_API_BASE

function randomColor() {
  return `hsl(${Math.floor(Math.random() * 360)}, 70%, 50%)`
}

async function fetchRank() {
  // Reset old values
  labels.value = []
  datasets.value = []
  latestRanks.value = {}

  const domainList = domain.value
    .split(',')
    .map((d) => d.trim().toLowerCase())
    .filter((d) => d.length > 0)

  for (const d of domainList) {
    await fetchSingleDomain(d)
  }

  renderChart()
}

async function fetchSingleDomain(d) {
  // Start loading...
  isLoading.value = true

  try {
    const response = await fetch(`${API_BASE}/ranking/${d}`)
    if (!response.ok) throw new Error('API Error')

    const data = await response.json()

    // Handle domain with no Tranco data
    if (!data.ranks || data.ranks.length === 0) {
      noRankMessages.value[d] =
        'This domain is currently outside Tranco’s Top 1M, but may appear occasionally in historical rankings.'
      return
    }

    noRankMessages.value = {}

    latestRanks.value[d] = data.ranks[data.ranks.length - 1]

    // Convert data
    const domainLabels = data.labels
    const domainRanks = data.ranks

    // initialize label if empty
    if (labels.value.length === 0) {
      labels.value = domainLabels
    }

    if (labels.value.length === 0) {
      labels.value = data.labels
    }

    const minLength = Math.min(labels.value.length, domainLabels.length)

    labels.value = labels.value.slice(-minLength)

    const alignedRanks = domainRanks.slice(-minLength)

    // Add dataset
    datasets.value.push({
      label: d,
      data: alignedRanks,
      borderColor: randomColor(),
      backgroundColor: randomColor(),
      pointRadius: 3,
      tension: 0.3,
    })

    isLoading.value = false
  } catch (err) {
    isLoading.value = false
    console.error('Error fetching', d, err)
  } finally {
    isLoading.value = false
  }
}

function renderChart() {
  if (chartInstance) chartInstance.destroy()

  chartInstance = new Chart(chartRef.value, {
    type: 'line',
    data: {
      labels: labels.value,
      datasets: datasets.value,
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
  <!-- Loading bar -->
  <div v-if="isLoading" class="loading-overlay">
    <div class="spinner"></div>
    <p>Fetching rank data...</p>
  </div>

  <div class="page">
    <h1 class="title">Get your domain rank</h1>
    <p class="subtitle">It uses tranco api to get ranking data under the hood.</p>

    <!-- SEARCH BAR -->
    <div class="search">
      <input v-model="domain" placeholder="google.com" class="input" />
      <button class="btn" @click="fetchRank" :disabled="isLoading">Get Rank</button>
    </div>

    <!-- LATEST RANK INFO -->
    <div v-if="latestRanks && Object.keys(latestRanks).length" class="latest">
      <p v-for="(rank, domain) in latestRanks" :key="domain">
        Latest rank for <strong>{{ domain }}</strong
        >: {{ rank }}
      </p>
    </div>

    <!-- NO RANK MESSAGE -->
    <div v-if="Object.keys(noRankMessages).length" class="no-rank">
      <p v-for="(msg, domain) in noRankMessages" :key="domain">
        <strong>{{ domain }}</strong
        >: {{ msg }}
      </p>
    </div>

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

.loading-overlay {
  position: fixed;
  inset: 0;
  background: rgba(255, 255, 255, 0.55);
  backdrop-filter: blur(1px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.spinner {
  width: 55px;
  height: 55px;
  border: 6px solid #ccc;
  border-top-color: #3b82f6;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.loading-text {
  margin-top: 12px;
  font-size: 18px;
  color: #333;
  font-weight: 600;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.no-rank {
  background: #f8fafc;
  border-left: 4px solid #f59e0b;
  padding: 12px 16px;
  border-radius: 6px;
  margin-bottom: 20px;
  color: #92400e;
  font-size: 15px;
}
</style>
