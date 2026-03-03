<template>
  <div class="panel-card power-load-chart" role="region" aria-label="实时电网负荷曲线">
    <div class="card-title">
      <div class="title-icon" aria-hidden="true"></div>
      <span>实时电网负荷曲线</span>
      <span class="title-unit">单位：万千瓦</span>
    </div>
    <div ref="chartRef" class="chart-body"
      role="img" aria-label="24小时电网负荷折线图，单位万千瓦"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref<HTMLElement>()
let chart: echarts.ECharts | null = null
let timer: ReturnType<typeof setInterval>

const BASE_LOADS = [
  420, 385, 355, 330, 318, 336, 495, 658,
  782, 856, 872, 845, 808, 825, 855, 874,
  892, 928, 882, 805, 724, 642, 562, 490
]

function getLoadData() {
  return BASE_LOADS.map(v => v + Math.round((Math.random() - 0.5) * 25))
}

function buildOption(loads: number[]) {
  const hours = Array.from({ length: 24 }, (_, i) => `${String(i).padStart(2, '0')}:00`)
  const currentHour = new Date().getHours()

  return {
    backgroundColor: 'transparent',
    grid: { top: 24, right: 12, bottom: 28, left: 52 },
    tooltip: {
      trigger: 'axis',
      backgroundColor: 'rgba(0, 15, 45, 0.92)',
      borderColor: 'rgba(0, 212, 255, 0.35)',
      borderWidth: 1,
      textStyle: { color: '#b0d4ff', fontSize: 11 },
      formatter: (params: echarts.TooltipComponentFormatterCallbackParams) => {
        if (Array.isArray(params)) {
          const p = params[0]
          return `${p.name}<br/><span style="color:#00d4ff">●</span> 负荷：<b style="color:#00d4ff">${p.value}</b> 万kW`
        }
        return ''
      }
    },
    xAxis: {
      type: 'category',
      data: hours,
      axisLine: { lineStyle: { color: 'rgba(0, 212, 255, 0.2)' } },
      axisLabel: { color: '#6aaabb', fontSize: 11, interval: 5 },
      axisTick: { show: false },
      splitLine: { show: false }
    },
    yAxis: {
      type: 'value',
      min: 200,
      axisLine: { show: false },
      axisLabel: { color: '#6aaabb', fontSize: 11 },
      splitLine: { lineStyle: { color: 'rgba(0, 100, 150, 0.15)', type: 'dashed' } }
    },
    series: [
      {
        type: 'line',
        data: loads,
        smooth: 0.4,
        symbol: 'none',
        lineStyle: { width: 2, color: '#00d4ff', shadowColor: 'rgba(0,212,255,0.4)', shadowBlur: 8 },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(0, 212, 255, 0.25)' },
            { offset: 1, color: 'rgba(0, 212, 255, 0.01)' }
          ])
        }
      },
      {
        type: 'scatter',
        data: [[currentHour, loads[currentHour]]],
        symbol: 'circle',
        symbolSize: 8,
        itemStyle: { color: '#00ff88', shadowColor: '#00ff88', shadowBlur: 12 }
      }
    ]
  }
}

function handleResize() { chart?.resize() }

onMounted(() => {
  if (!chartRef.value) return
  chart = echarts.init(chartRef.value)
  chart.setOption(buildOption(getLoadData()))
  timer = setInterval(() => {
    chart?.setOption(buildOption(getLoadData()))
  }, 5000)
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  clearInterval(timer)
  window.removeEventListener('resize', handleResize)
  chart?.dispose()
})
</script>

<style scoped>
.power-load-chart {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.chart-body {
  flex: 1;
  min-height: 0;
}
</style>
