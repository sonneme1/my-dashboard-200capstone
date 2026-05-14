<template>
  <v-app>
    <v-app-bar color="primary" dark flat>
      <v-toolbar-title class="font-weight-bold">FastForward Logistics</v-toolbar-title>
      <v-spacer />
      <v-select
        v-model="selectedRegion"
        :items="regionOptions"
        label="Region"
        variant="underlined"
        class="mx-2"
        style="max-width: 160px"
        hide-details
      />
      <v-select
        v-model="selectedPeriod"
        :items="periodOptions"
        label="Time Period"
        variant="underlined"
        class="mx-2"
        style="max-width: 180px"
        hide-details
      />
    </v-app-bar>
    <v-main>
      <v-container fluid class="py-6">
        <!-- Summary Cards -->
        <v-row class="mb-6" align="stretch">
          <v-col cols="12" sm="6" md="3" v-for="card in summaryCards" :key="card.title" class="d-flex">
            <v-card class="pa-4 flex-grow-1 d-flex flex-column justify-space-between summary-card" elevation="2" :color="card.trend >= 0 ? 'surface' : 'surface-variant'">
              <div>
                <div class="d-flex align-center justify-space-between mb-2">
                  <span class="text-h6 font-weight-bold">{{ card.title }}</span>
                  <v-icon :color="card.trend > 0 ? 'success' : card.trend < 0 ? 'error' : 'grey'">
                    {{ card.trend > 0 ? 'mdi-arrow-up' : card.trend < 0 ? 'mdi-arrow-down' : 'mdi-minus' }}
                  </v-icon>
                </div>
                <div class="text-h4 font-weight-bold mb-1">{{ card.value }}</div>
              </div>
              <div class="text-caption" :class="card.trend > 0 ? 'text-success' : card.trend < 0 ? 'text-error' : ''">
                {{ card.trend > 0 ? '+' : '' }}{{ card.trendPercent }}% vs prev
              </div>
            </v-card>
          </v-col>
        </v-row>

        <!-- Charts Row -->
        <v-row align="stretch">
          <v-col cols="12" md="6" class="d-flex">
            <v-card class="pa-4 mb-6 flex-grow-1 d-flex flex-column chart-row-card" elevation="2">
              <div class="text-h6 font-weight-bold mb-2">Shipment Volume Trend</div>
              <Bar :data="shipmentVolumeData" :options="barChartOptions" />
            </v-card>
          </v-col>
          <v-col cols="12" md="6" class="d-flex">
            <v-card class="pa-4 mb-6 flex-grow-1 d-flex flex-column chart-row-card" elevation="2">
              <div class="text-h6 font-weight-bold mb-2">On-Time Delivery Rate</div>
              <Line :data="onTimeDeliveryData" :options="lineChartOptions" />
            </v-card>
          </v-col>
        </v-row>

        <v-row align="stretch">
          <v-col cols="12" md="6" class="d-flex">
            <v-card class="pa-4 mb-6 flex-grow-1 d-flex flex-column chart-row-card" elevation="2">
              <div class="text-h6 font-weight-bold mb-2">Regional Performance</div>
              <div style="position:relative; height:340px; width:100%;">
                <Bar :data="regionalPerformanceData" :options="barChartOptions" />
              </div>
            </v-card>
          </v-col>
          <v-col cols="12" md="6" class="d-flex">
            <v-card class="pa-4 mb-6 flex-grow-1 d-flex flex-column chart-row-card" elevation="2">
              <div class="text-h6 font-weight-bold mb-2">Open Exceptions</div>
              <Doughnut :data="exceptionsData" :options="doughnutOptions" />
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { Bar, Line, Doughnut } from 'vue-chartjs'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  LineElement,
  CategoryScale,
  LinearScale,
  PointElement,
  ArcElement,
} from 'chart.js'
import metricsRaw from '../data/metrics.json'

ChartJS.register(Title, Tooltip, Legend, BarElement, LineElement, CategoryScale, LinearScale, PointElement, ArcElement)

const regionOptions = [
  { title: 'All', value: 'All' },
  ...metricsRaw.regions.map((r: string) => ({ title: r, value: r })),
]
const periodOptions = [
  { title: 'Last 30 Days', value: '30d' },
  { title: 'Last 60 Days', value: '60d' },
  { title: 'Last 6 Months', value: '6m' },
  { title: 'Last Year', value: '1y' },
]

const selectedRegion = ref('All')
const selectedPeriod = ref('1y')

const months = metricsRaw.months
const regions = metricsRaw.regions
const metrics = metricsRaw.metrics

function getPeriodMonths(period: string) {
  if (period === '30d') return months.slice(-1)
  if (period === '60d') return months.slice(-2)
  if (period === '6m') return months.slice(-6)
  return months // 1y
}

const filteredMetrics = computed(() => {
  const periodMonths = getPeriodMonths(selectedPeriod.value)
  return metrics.filter((m: any) =>
    periodMonths.includes(m.month) &&
    (selectedRegion.value === 'All' || m.region === selectedRegion.value)
  )
})

const summaryCards = computed(() => {
  // Shipment Volume
  const periodMonths = getPeriodMonths(selectedPeriod.value)
  const prevMonths = months.slice(periodMonths.length * -2, periodMonths.length * -1)
  const current = filteredMetrics.value
  const prev = metrics.filter((m: any) => prevMonths.includes(m.month) && (selectedRegion.value === 'All' || m.region === selectedRegion.value))

  // Helper to sum a field
  const sum = (arr: any[], fn: (m: any) => number) => arr.reduce((a, b) => a + fn(b), 0)

  const totalShipments = sum(current, m => m.shipments.total)
  const prevShipments = sum(prev, m => m.shipments.total) || 1
  const shipmentTrend = ((totalShipments - prevShipments) / prevShipments) * 100

  const onTimeRate = current.length ? (sum(current, m => m.onTimeDelivery.rate) / current.length) : 0
  const prevOnTimeRate = prev.length ? (sum(prev, m => m.onTimeDelivery.rate) / prev.length) : 0
  const onTimeTrend = ((onTimeRate - prevOnTimeRate) / (prevOnTimeRate || 1)) * 100

  const openExceptions = sum(current, m => m.exceptions.open)
  const prevExceptions = sum(prev, m => m.exceptions.open) || 1
  const exceptionsTrend = ((openExceptions - prevExceptions) / prevExceptions) * 100

  return [
    {
      title: 'Total Shipments',
      value: totalShipments.toLocaleString(),
      trend: shipmentTrend,
      trendPercent: shipmentTrend.toFixed(1),
    },
    {
      title: 'On-Time Delivery Rate',
      value: (onTimeRate * 100).toFixed(1) + '%',
      trend: onTimeTrend,
      trendPercent: onTimeTrend.toFixed(1),
    },
    {
      title: 'Open Exceptions',
      value: openExceptions,
      trend: exceptionsTrend,
      trendPercent: exceptionsTrend.toFixed(1),
    },
    {
      title: 'Late Shipments',
      value: sum(current, m => m.onTimeDelivery.lateCount),
      trend: 0,
      trendPercent: '0.0',
    },
  ]
})

// Shipment Volume Trend (Bar)
const shipmentVolumeData = computed(() => {
  const periodMonths = getPeriodMonths(selectedPeriod.value)
  const labels = periodMonths.map(m => m.slice(5) + '/25')
  const data = regions.map(region => {
    return {
      label: region,
      backgroundColor: regionColor(region),
      data: periodMonths.map(month => {
        const metric = metrics.find((m: any) => m.month === month && m.region === region)
        return metric ? metric.shipments.total : 0
      })
    }
  })
  return { labels, datasets: data }
})

// On-Time Delivery Rate (Line)
const onTimeDeliveryData = computed(() => {
  const periodMonths = getPeriodMonths(selectedPeriod.value)
  return {
    labels: periodMonths.map(m => m.slice(5) + '/25'),
    datasets: [
      {
        label: 'On-Time Rate',
        borderColor: '#4caf50',
        backgroundColor: 'rgba(76,175,80,0.2)',
        data: periodMonths.map(month => {
          const ms = regions.map(region => metrics.find((m: any) => m.month === month && m.region === region)).filter(Boolean)
          if (!ms.length) return 0
          return (ms.reduce((a, b) => a + b.onTimeDelivery.rate, 0) / ms.length) * 100
        })
      }
    ]
  }
})

// Regional Performance (Bar)
const regionalPerformanceData = computed(() => {
  const periodMonths = getPeriodMonths(selectedPeriod.value)
  return {
    labels: regions,
    datasets: [
      {
        label: 'Shipment Volume',
        backgroundColor: regions.map(region => regionColor(region)),
        data: regions.map(region => {
          return metrics.filter((m: any) => periodMonths.includes(m.month) && m.region === region).reduce((a, b) => a + b.shipments.total, 0)
        })
      },
      {
        label: 'On-Time Rate',
        backgroundColor: regions.map(region => regionColor(region, true)),
        data: regions.map(region => {
          const ms = metrics.filter((m: any) => periodMonths.includes(m.month) && m.region === region)
          if (!ms.length) return 0
          return (ms.reduce((a, b) => a + b.onTimeDelivery.rate, 0) / ms.length) * 100
        })
      }
    ]
  }
})

// Open Exceptions (Doughnut)
const exceptionsData = computed(() => {
  const periodMonths = getPeriodMonths(selectedPeriod.value)
  const filtered = metrics.filter((m: any) => periodMonths.includes(m.month) && (selectedRegion.value === 'All' || m.region === selectedRegion.value))
  const sumType = (type: string) => filtered.reduce((a, b) => a + (b.exceptions.byType[type] || 0), 0)
  const types = ['damaged', 'delayed', 'lost', 'customsHold']
  return {
    labels: types.map(t => t.charAt(0).toUpperCase() + t.slice(1)),
    datasets: [
      {
        backgroundColor: ['#ff9800', '#1976d2', '#e53935', '#9c27b0'],
        data: types.map(sumType)
      }
    ]
  }
})

function regionColor(region: string, faded = false) {
  const colors: Record<string, string> = {
    North: faded ? '#90caf9' : '#1976d2',
    South: faded ? '#a5d6a7' : '#388e3c',
    East: faded ? '#ffe082' : '#fbc02d',
    West: faded ? '#b39ddb' : '#7e57c2',
  }
  return colors[region] || '#bdbdbd'
}

const barChartOptions = {
  responsive: true,
  plugins: {
    legend: { display: true },
    tooltip: { enabled: true },
  },
  scales: {
    x: { grid: { color: '#333' } },
    y: { grid: { color: '#333' } },
  },
}

const lineChartOptions = {
  responsive: true,
  plugins: {
    legend: { display: true },
    tooltip: { enabled: true },
  },
  elements: {
    line: { tension: 0.3 },
    point: { radius: 4 },
  },
  scales: {
    x: { grid: { color: '#333' } },
    y: { grid: { color: '#333' } },
  },
}

const doughnutOptions = {
  responsive: true,
  plugins: {
    legend: { display: true },
    tooltip: { enabled: true },
  },
}
</script>

.chart-row-card {
  min-height: 340px;
}
.chart-row-card .chartjs-render-monitor,
.chart-row-card canvas {
  height: 100% !important;
  max-height: 100% !important;
  width: 100% !important;
  max-width: 100% !important;
  aspect-ratio: unset !important;
}
.summary-card {
  min-height: 160px;
}
.v-application {
  background: #16171d;
  color: #f3f4f6;
}
.v-card {
  border-radius: 18px;
}
.text-success {
  color: #4caf50 !important;
}
.text-error {
  color: #e53935 !important;
}

