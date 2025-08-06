<template>
  <div>
    <div class="flex items-center justify-between mb-4">
      <h2 class="text-lg font-medium text-gray-900">Signups Overview</h2>
      <div class="flex space-x-4">
        <button
          @click="chartType = 'day'"
          class="px-3 py-1 text-sm rounded-md"
          :class="{
            'bg-tena-green text-white': chartType === 'day',
            'bg-gray-100 text-gray-700 hover:bg-gray-200': chartType !== 'day',
          }"
        >
          By Day
        </button>
        <button
          @click="chartType = 'source'"
          class="px-3 py-1 text-sm rounded-md"
          :class="{
            'bg-tena-green text-white': chartType === 'source',
            'bg-gray-100 text-gray-700 hover:bg-gray-200':
              chartType !== 'source',
          }"
        >
          By Source
        </button>
      </div>
    </div>

    <div class="h-64">
      <canvas ref="chartCanvas"></canvas>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue';
import Chart from 'chart.js/auto';

const props = defineProps({
  users: {
    type: Array,
    required: true,
  },
});

const chartType = ref('day');
const chartCanvas = ref(null);
let chartInstance = null;

const prepareDayData = () => {
  const last30Days = Array(30)
    .fill(0)
    .map((_, i) => {
      const date = new Date();
      date.setDate(date.getDate() - (29 - i));
      return date.toISOString().split('T')[0];
    });

  const signupsByDay = last30Days.reduce((acc, day) => {
    acc[day] = props.users.filter(
      (user) => user.signupDate.split('T')[0] === day
    ).length;
    return acc;
  }, {});

  return {
    labels: last30Days.map((date) =>
      new Date(date).toLocaleDateString('en-US', {
        month: 'short',
        day: 'numeric',
      })
    ),
    data: last30Days.map((date) => signupsByDay[date] || 0),
  };
};

const prepareSourceData = () => {
  const sources = [...new Set(props.users.map((user) => user.source))];
  const signupsBySource = sources.reduce((acc, source) => {
    acc[source] = props.users.filter((user) => user.source === source).length;
    return acc;
  }, {});

  return {
    labels: sources,
    data: sources.map((source) => signupsBySource[source] || 0),
  };
};

const renderChart = () => {
  if (chartInstance) {
    chartInstance.destroy();
  }

  const ctx = chartCanvas.value.getContext('2d');
  const { labels, data } =
    chartType.value === 'day' ? prepareDayData() : prepareSourceData();

  chartInstance = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: labels,
      datasets: [
        {
          label:
            chartType.value === 'day' ? 'Signups per day' : 'Signups by source',
          data: data,
          backgroundColor: '#2e8b57',
          borderRadius: 4,
          borderSkipped: false,
        },
      ],
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        y: {
          beginAtZero: true,
          ticks: {
            precision: 0,
          },
        },
      },
      plugins: {
        legend: {
          display: false,
        },
      },
    },
  });
};

watch([() => props.users, chartType], () => {
  renderChart();
});

onMounted(() => {
  renderChart();
});

onUnmounted(() => {
  if (chartInstance) {
    chartInstance.destroy();
  }
});
</script>
