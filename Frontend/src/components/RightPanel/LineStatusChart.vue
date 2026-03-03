<template>
  <div class="panel-card line-status-chart" role="region" aria-label="输电线路负荷率">
    <div class="card-title">
      <div class="title-icon" aria-hidden="true"></div>
      <span>输电线路负荷率</span>
      <span class="title-unit">实时监测</span>
    </div>
    <div ref="chartRef" class="chart-body"
      role="img" aria-label="输电线路负荷率水平柱状图"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref<HTMLElement>()
let chart: echarts.ECharts | null = null
let timer: ReturnType<typeof setInterval>

const LINES = [
  '昌顺220kV', '顺通110kV', '通兴220kV',
  '兴房110kV', '房密110kV', '密延110kV', '延石500kV'
]

function getLoadRates() {
  return LINES.map(() => Math.round(45 + Math.random() * 50))
}

function buildOption(rates: number[]) {
  const colors = rates.map(r =>
    r > 90 ? '#ff4444' : r > 75 ? '#ff9500' : '#00d4ff'
  )

  return {
    backgroundColor: 'transparent',
    grid: { top: 8, right: 60, bottom: 8, left: 72, containLabel: false },
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'none' },
      backgroundColor: 'rgba(0, 15, 45, 0.92)',
      borderColor: 'rgba(0, 212, 255, 0.35)',
      textStyle: { color: '#b0d4ff', fontSize: 11 },
      formatter: (params: echarts.TooltipComponentFormatterCallbackParams) => {
        if (Array.isArray(params)) {
          const p = params[0]
          return `${p.name}<br/>负荷率：<b style="color:#00d4ff">${p.value}%</b>`
        }
        return ''
      }
    },
    xAxis: {
      type: 'value',
      max: 100,
      axisLine: { show: false },
      axisLabel: { show: false },
      splitLine: { lineStyle: { color: 'rgba(0, 80, 120, 0.2)', type: 'dashed' } }
    },
    yAxis: {
      type: 'category',
      data: LINES,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: { color: '#7db8d4', fontSize: 11 }
    },
    series: [{
      type: 'bar',
      data: rates.map((v, i) => ({
        value: v,
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [
            { offset: 0, color: colors[i] + '55' },
            { offset: 1, color: colors[i] }
          ]),
          borderRadius: [0, 2, 2, 0]
        }
      })),
      barWidth: 10,
      label: {
        show: true,
        position: 'right',
        color: '#7db8d4',
        fontSize: 11,
        formatter: '{c}%'
      }
    }]
  }
}

function handleResize() { chart?.resize() }

onMounted(() => {
  if (!chartRef.value) return
  chart = echarts.init(chartRef.value)
  chart.setOption(buildOption(getLoadRates()))
  timer = setInterval(() => {
    chart?.setOption(buildOption(getLoadRates()))
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
.line-status-chart {
  flex: 0 0 auto;
  height: 210px;
  display: flex;
  flex-direction: column;
}

.chart-body {
  flex: 1;
  min-height: 0;
}
</style>
