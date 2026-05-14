<template>
  <v-app>
    <v-main>
      <v-container>
        <h1>Welcome to My Dashboard</h1>
        <Bar :data="chartData" :options="chartOptions" />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { Bar } from 'vue-chartjs'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
} from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

const chartData = {
  labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
  datasets: [
    {
      label: 'Sales',
      backgroundColor: '#1976D2',
      data: [40, 55, 65, 59, 80],
    },
  ],
}

const chartOptions = {
  responsive: true,
  plugins: {
    legend: {
      display: true,
    },
  },
}
</script>
