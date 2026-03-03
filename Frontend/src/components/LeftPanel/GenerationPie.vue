<template>
  <div class="panel-card generation-pie" role="region" aria-label="发电能源结构">
    <div class="card-title">
      <div class="title-icon" aria-hidden="true"></div>
      <span>发电能源结构</span>
      <span class="title-unit">单位：亿千瓦时</span>
    </div>
    <div class="pie-body">
      <div ref="chartRef" class="pie-chart"
        role="img"
        :aria-label="legendData.map(d => `${d.name} ${d.value}亿千瓦时 占比${d.pct}%`).join('，')"></div>
      <div class="legend-list" role="list" aria-label="能源结构图例">
        <div v-for="item in legendData" :key="item.name" class="legend-item" role="listitem"
          :aria-label="`${item.name}：${item.value} 亿千瓦时，占比 ${item.pct}%`">
          <span class="legend-dot" aria-hidden="true"
            :style="{ background: item.color, boxShadow: `0 0 6px ${item.color}` }"></span>
          <span class="legend-name">{{ item.name }}</span>
          <span class="legend-value" :style="{ color: item.color }" aria-hidden="true">{{ item.value }}</span>
          <span class="legend-pct" aria-hidden="true">{{ item.pct }}%</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'

interface GenItem { name: string; value: number; color: string; pct: string }

const chartRef = ref<HTMLElement>()
let chart: echarts.ECharts | null = null

const COLORS = ['#00d4ff', '#00ff88', '#0080ff', '#ff9500', '#7b68ee', '#ff4444']
const SOURCES = [
  { name: '火力发电', baseVal: 9.2 },
  { name: '水力发电', baseVal: 5.8 },
  { name: '核能发电', baseVal: 3.4 },
  { name: '风力发电', baseVal: 4.1 },
  { name: '光伏发电', baseVal: 2.9 },
  { name: '其他新能源', baseVal: 1.2 }
]

const legendData = ref<GenItem[]>([])

function generateData(): GenItem[] {
  const vals = SOURCES.map(s => parseFloat((s.baseVal + (Math.random() - 0.5) * 0.3).toFixed(2)))
  const total = vals.reduce((a, b) => a + b, 0)
  return SOURCES.map((s, i) => ({
    name: s.name,
    value: vals[i],
    color: COLORS[i],
    pct: ((vals[i] / total) * 100).toFixed(1)
  }))
}

function buildOption(data: GenItem[]) {
  return {
    backgroundColor: 'transparent',
    series: [{
      type: 'pie',
      radius: ['38%', '68%'],
      center: ['50%', '50%'],
      data: data.map(d => ({ name: d.name, value: d.value })),
      itemStyle: {
        color: (params: { dataIndex: number }) => COLORS[params.dataIndex],
        borderColor: '#050a1a',
        borderWidth: 2
      },
      label: { show: false },
      emphasis: {
        itemStyle: {
          shadowBlur: 20,
          shadowColor: 'rgba(0, 212, 255, 0.5)'
        }
      }
    }]
  }
}

let timer: ReturnType<typeof setInterval>

function handleResize() { chart?.resize() }

onMounted(() => {
  if (!chartRef.value) return
  chart = echarts.init(chartRef.value)
  legendData.value = generateData()
  chart.setOption(buildOption(legendData.value))
  timer = setInterval(() => {
    legendData.value = generateData()
    chart?.setOption(buildOption(legendData.value))
  }, 8000)
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  clearInterval(timer)
  window.removeEventListener('resize', handleResize)
  chart?.dispose()
})
</script>

<style scoped>
.generation-pie {
  flex: 0 0 auto;
  height: 175px;
  display: flex;
  flex-direction: column;
}

.pie-body {
  flex: 1;
  display: flex;
  align-items: center;
  min-height: 0;
  padding: 6px 8px;
  gap: 8px;
}

.pie-chart {
  width: 120px;
  height: 120px;
  flex-shrink: 0;
}

.legend-list {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 12px;
}

.legend-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  flex-shrink: 0;
}

.legend-name {
  flex: 1;
  color: var(--text-secondary);
  white-space: nowrap;
}

.legend-value {
  font-family: 'Courier New', monospace;
  font-size: 12px;
  font-weight: 600;
  min-width: 30px;
  text-align: right;
}

.legend-pct {
  color: var(--text-muted);
  min-width: 32px;
  text-align: right;
}
</style>
