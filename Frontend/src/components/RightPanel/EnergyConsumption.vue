<template>
  <div class="panel-card energy-consumption" role="region" aria-label="各区用电量统计">
    <div class="card-title">
      <div class="title-icon" aria-hidden="true"></div>
      <span>各区用电量统计</span>
      <span class="title-unit">单位：万kWh</span>
    </div>
    <div ref="chartRef" class="chart-body"
      role="img" aria-label="各区用电量柱状图，单位万千瓦时"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref<HTMLElement>()
let chart: echarts.ECharts | null = null
let timer: ReturnType<typeof setInterval>

const DISTRICTS = ['朝阳', '海淀', '丰台', '西城', '东城', '石景山', '顺义', '大兴']
const BASE_VALUES = [420, 388, 310, 195, 172, 135, 285, 265]

function getConsumption() {
  return BASE_VALUES.map(v => Math.round(v + (Math.random() - 0.5) * 30))
}

function buildOption(values: number[]) {
  return {
    backgroundColor: 'transparent',
    grid: { top: 14, right: 12, bottom: 30, left: 36 },
    tooltip: {
      trigger: 'axis',
      backgroundColor: 'rgba(0, 15, 45, 0.92)',
      borderColor: 'rgba(0, 212, 255, 0.35)',
      textStyle: { color: '#b0d4ff', fontSize: 11 }
    },
    xAxis: {
      type: 'category',
      data: DISTRICTS,
      axisLine: { lineStyle: { color: 'rgba(0, 212, 255, 0.2)' } },
      /* 颜色从 #5a8a9a 提升至 #7db8d4（text-secondary），字号从 9 提升至 11 */
      axisLabel: { color: '#7db8d4', fontSize: 11 },
      axisTick: { show: false }
    },
    yAxis: {
      type: 'value',
      axisLine: { show: false },
      /* 颜色从 #3a6a7a 提升至 #6aaabb（text-muted 新值），字号从 9 提升至 11 */
      axisLabel: { color: '#6aaabb', fontSize: 11 },
      splitLine: { lineStyle: { color: 'rgba(0, 80, 120, 0.15)', type: 'dashed' } }
    },
    series: [{
      type: 'bar',
      data: values.map((v, i) => ({
        value: v,
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: i < 3 ? '#00d4ff' : '#0080ff' },
            { offset: 1, color: i < 3 ? '#004488' : '#003366' }
          ]),
          borderRadius: [2, 2, 0, 0]
        }
      })),
      barWidth: '60%',
      emphasis: {
        itemStyle: {
          shadowBlur: 10,
          shadowColor: 'rgba(0, 212, 255, 0.4)'
        }
      }
    }]
  }
}

function handleResize() { chart?.resize() }

onMounted(() => {
  if (!chartRef.value) return
  chart = echarts.init(chartRef.value)
  chart.setOption(buildOption(getConsumption()))
  timer = setInterval(() => {
    chart?.setOption(buildOption(getConsumption()))
  }, 6000)
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  clearInterval(timer)
  window.removeEventListener('resize', handleResize)
  chart?.dispose()
})
</script>

<style scoped>
.energy-consumption {
  flex: 0 0 auto;
  height: 185px;
  display: flex;
  flex-direction: column;
}

.chart-body {
  flex: 1;
  min-height: 0;
}
</style>
