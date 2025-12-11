<template>
  <div class="chart-container">
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, onBeforeUnmount } from 'vue';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  LineController,
  Filler,
  Title,
  Tooltip,
  Legend,
  type ChartOptions
} from 'chart.js';

// 注册所有必需的组件（Chart.js v4 需要显式注册控制器和插件）
ChartJS.register(
  LineController,  // 必须先注册控制器
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Filler,  // 注册 Filler 插件以支持 fill 选项
  Title,
  Tooltip,
  Legend
);

interface Props {
  labels: string[];
  datasets: Array<{
    label: string;
    data: number[];
    borderColor?: string;
    backgroundColor?: string;
    tension?: number;
  }>;
  options?: ChartOptions<'line'>;
}

const props = defineProps<Props>();

const chartCanvas = ref<HTMLCanvasElement | null>(null);
let chartInstance: ChartJS<'line'> | null = null;

const initChart = () => {
  if (!chartCanvas.value) return;
  
  // 如果图表已存在，先销毁
  if (chartInstance) {
    chartInstance.destroy();
    chartInstance = null;
  }
  
  // 如果没有数据，不创建图表
  if (!props.labels || props.labels.length === 0 || !props.datasets || props.datasets.length === 0) {
    console.log('[LineChart] 没有数据，跳过图表创建');
    return;
  }
  
  console.log('[LineChart] 创建图表，labels:', props.labels, 'datasets:', props.datasets);
  
  // 处理渐变背景色（如果是函数，需要在图表创建后处理）
  const processedDatasets = [...props.datasets];
  
  chartInstance = new ChartJS(chartCanvas.value, {
    type: 'line',
    data: {
      labels: props.labels,
      datasets: processedDatasets
    },
    options: props.options || {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          position: 'top' as const,
        },
        title: {
          display: false,
        },
      },
      scales: {
        y: {
          beginAtZero: true,
        },
      },
    },
  });
  
  // 如果数据集中有渐变函数，在图表创建后处理
  props.datasets.forEach((dataset, index) => {
    if (typeof dataset.backgroundColor === 'function' && chartInstance) {
      const ctx = chartInstance.ctx;
      const chartArea = chartInstance.chartArea;
      if (ctx && chartArea) {
        const gradient = ctx.createLinearGradient(0, chartArea.top, 0, chartArea.bottom);
        gradient.addColorStop(0, 'rgba(74, 111, 165, 0.3)');
        gradient.addColorStop(1, 'rgba(74, 111, 165, 0.05)');
        chartInstance.data.datasets[index].backgroundColor = gradient;
        chartInstance.update();
      }
    }
  });
};

onMounted(() => {
  // 延迟初始化，确保 DOM 已渲染
  setTimeout(() => {
    initChart();
  }, 100);
});

watch(
  () => [props.labels, props.datasets],
  () => {
    console.log('[LineChart] Props 变化，labels:', props.labels, 'datasets:', props.datasets);
    if (chartInstance) {
      // 如果数据为空，销毁图表
      if (!props.labels || props.labels.length === 0 || !props.datasets || props.datasets.length === 0) {
        chartInstance.destroy();
        chartInstance = null;
        return;
      }
      chartInstance.data.labels = props.labels;
      chartInstance.data.datasets = props.datasets;
      chartInstance.update();
    } else {
      // 如果图表不存在但有数据，重新初始化
      if (props.labels && props.labels.length > 0 && props.datasets && props.datasets.length > 0) {
        initChart();
      }
    }
  },
  { deep: true, immediate: true }
);

onBeforeUnmount(() => {
  if (chartInstance) {
    chartInstance.destroy();
  }
});
</script>

<style scoped>
.chart-container {
  width: 100%;
  height: 100%;
  min-height: 300px;
  position: relative;
}
</style>

