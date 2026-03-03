<template>
  <div class="map-view panel-card">
    <div class="map-header">
      <div class="card-title" style="border-bottom: none; padding-bottom: 0;">
        <div class="title-icon"></div>
        <span>北京电网态势感知图</span>
        <div class="map-controls">
          <span
            v-for="mode in mapModes"
            :key="mode.key"
            class="ctrl-btn"
            :class="{ active: activeMode === mode.key }"
            @click="switchMode(mode.key)"
          >{{ mode.label }}</span>
        </div>
      </div>
      <div class="map-stats-row">
        <div v-for="stat in mapStats" :key="stat.label" class="mstat">
          <span class="mstat-val tech-number" :style="{ color: stat.color }">{{ stat.value }}</span>
          <span class="mstat-label">{{ stat.label }}</span>
        </div>
      </div>
    </div>

    <div class="map-container-3d">
      <div class="map-scan"></div>
      <div ref="chartRef" class="map-echarts"></div>
      <div class="grid-overlay"></div>
      <!-- 3D badge -->
      <div class="badge-3d">3D</div>
    </div>

    <!-- Legend -->
    <div class="map-legend">
      <div class="legend-item">
        <span class="l-dot" style="background:#00ff88; box-shadow: 0 0 6px #00ff88;"></span>
        <span>500kV变电站</span>
      </div>
      <div class="legend-item">
        <span class="l-dot" style="background:#00d4ff; box-shadow: 0 0 6px #00d4ff;"></span>
        <span>220kV变电站</span>
      </div>
      <div class="legend-item">
        <span class="l-dot" style="background:#7b68ee; box-shadow: 0 0 6px #7b68ee;"></span>
        <span>110kV变电站</span>
      </div>
      <div class="legend-item">
        <span class="l-line" style="background: linear-gradient(90deg, #ff4444, #ff9500);"></span>
        <span>高负荷线路</span>
      </div>
      <div class="legend-item">
        <span class="l-line" style="background: linear-gradient(90deg, #00d4ff, #0080ff);"></span>
        <span>正常线路</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as echarts from 'echarts'
import 'echarts-gl'

const chartRef = ref<HTMLElement>()
let chart: echarts.ECharts | null = null
let timer: ReturnType<typeof setInterval>
let mapLoaded = false
const activeMode = ref('grid')

const mapModes = [
  { key: 'grid', label: '电网' },
  { key: 'load', label: '负荷' },
  { key: 'alarm', label: '告警' }
]

const mapStats = ref([
  { label: '变电站', value: '312', color: '#00d4ff' },
  { label: '输电线路', value: '856', color: '#00ff88' },
  { label: '供电面积', value: '16807', color: '#7b68ee' },
  { label: '供电用户', value: '1083万', color: '#ff9500' }
])

const SUBSTATIONS_500 = [
  { name: '顺义500kV', coord: [116.65, 40.13] },
  { name: '房山500kV', coord: [116.13, 39.73] },
  { name: '延庆500kV', coord: [115.97, 40.47] }
]

const SUBSTATIONS_220 = [
  { name: '昌平220kV', coord: [116.23, 40.22] },
  { name: '通州220kV', coord: [116.66, 39.91] },
  { name: '大兴220kV', coord: [116.33, 39.72] },
  { name: '密云220kV', coord: [116.84, 40.38] },
  { name: '平谷220kV', coord: [117.12, 40.14] },
  { name: '怀柔220kV', coord: [116.64, 40.32] }
]

const SUBSTATIONS_110 = [
  { name: '海淀110kV', coord: [116.30, 40.05] },
  { name: '朝阳110kV', coord: [116.48, 39.95] },
  { name: '丰台110kV', coord: [116.28, 39.86] },
  { name: '石景山110kV', coord: [116.22, 39.91] },
  { name: '东城110kV', coord: [116.42, 39.92] },
  { name: '西城110kV', coord: [116.36, 39.92] },
  { name: '门头沟110kV', coord: [116.10, 39.94] }
]

const TRANSMISSION_LINES = [
  { from: [116.65, 40.13], to: [116.23, 40.22], load: 65 },
  { from: [116.65, 40.13], to: [116.84, 40.38], load: 78 },
  { from: [116.13, 39.73], to: [116.28, 39.86], load: 82 },
  { from: [116.13, 39.73], to: [116.33, 39.72], load: 55 },
  { from: [115.97, 40.47], to: [116.23, 40.22], load: 48 },
  { from: [116.23, 40.22], to: [116.30, 40.05], load: 71 },
  { from: [116.23, 40.22], to: [116.64, 40.32], load: 60 },
  { from: [116.66, 39.91], to: [116.48, 39.95], load: 88 },
  { from: [116.66, 39.91], to: [117.12, 40.14], load: 45 },
  { from: [116.33, 39.72], to: [116.28, 39.86], load: 67 },
  { from: [116.30, 40.05], to: [116.36, 39.92], load: 74 },
  { from: [116.30, 40.05], to: [116.22, 39.91], load: 58 },
  { from: [116.48, 39.95], to: [116.42, 39.92], load: 91 }
]

// District load data for bar height in 3D map
const DISTRICT_LOADS: Record<string, number> = {
  '东城区': 172, '西城区': 195, '朝阳区': 420, '丰台区': 310,
  '石景山区': 135, '海淀区': 388, '门头沟区': 88, '房山区': 145,
  '通州区': 215, '顺义区': 285, '昌平区': 248, '大兴区': 265,
  '怀柔区': 92, '平谷区': 78, '密云区': 105, '延庆区': 65
}

function getLineColor(load: number) {
  if (load > 85) return '#ff4444'
  if (load > 70) return '#ff9500'
  return '#00d4ff'
}

function getDistrictColor(load: number, mode: string) {
  if (mode === 'alarm') {
    if (load > 350) return '#ff4444'
    if (load > 200) return '#ff9500'
    return '#00ff88'
  }
  if (mode === 'load') {
    if (load > 350) return new echarts.graphic.LinearGradient(0, 0, 0, 1, [
      { offset: 0, color: 'rgba(255, 68, 68, 0.95)' },
      { offset: 1, color: 'rgba(100, 10, 10, 0.6)' }
    ])
    if (load > 200) return new echarts.graphic.LinearGradient(0, 0, 0, 1, [
      { offset: 0, color: 'rgba(255, 149, 0, 0.95)' },
      { offset: 1, color: 'rgba(80, 40, 0, 0.6)' }
    ])
    return new echarts.graphic.LinearGradient(0, 0, 0, 1, [
      { offset: 0, color: 'rgba(0, 212, 255, 0.9)' },
      { offset: 1, color: 'rgba(0, 50, 100, 0.6)' }
    ])
  }
  // grid mode
  return new echarts.graphic.LinearGradient(0, 0, 0, 1, [
    { offset: 0, color: 'rgba(0, 180, 255, 0.95)' },
    { offset: 1, color: 'rgba(0, 40, 100, 0.55)' }
  ])
}

function buildOption() {
  const mode = activeMode.value

  // Build bar3D data from district loads
  const bar3DData = Object.entries(DISTRICT_LOADS).map(([name, baseLoad]) => {
    const load = baseLoad + Math.round((Math.random() - 0.5) * 20)
    return {
      name,
      value: load,
      itemStyle: { color: getDistrictColor(load, mode) }
    }
  })

  // Scatter3D for substations
  const scatter500 = SUBSTATIONS_500.map(s => ({
    name: s.name,
    value: [...s.coord, 80],
    symbolSize: 14,
    itemStyle: { color: '#00ff88' },
    label: { show: true, formatter: (p: { name: string }) => p.name, color: '#00ff88', fontSize: 9, distance: 5 }
  }))

  const scatter220 = SUBSTATIONS_220.map(s => ({
    name: s.name,
    value: [...s.coord, 60],
    symbolSize: 9,
    itemStyle: { color: '#00d4ff' },
    label: { show: false }
  }))

  const scatter110 = SUBSTATIONS_110.map(s => ({
    name: s.name,
    value: [...s.coord, 45],
    symbolSize: 7,
    itemStyle: { color: '#7b68ee' },
    label: { show: false }
  }))

  // Lines3D for transmission
  const lines3DData = TRANSMISSION_LINES.map(line => ({
    coords: [[...line.from, 20], [...line.to, 20]],
    lineStyle: {
      color: getLineColor(line.load + Math.round((Math.random() - 0.5) * 10)),
      width: 2,
      opacity: 0.85
    }
  }))

  return {
    backgroundColor: 'transparent',
    tooltip: {
      show: true,
      backgroundColor: 'rgba(0, 10, 35, 0.95)',
      borderColor: 'rgba(0, 212, 255, 0.6)',
      borderWidth: 1,
      padding: [8, 12],
      textStyle: { color: '#ffffff', fontSize: 13 },
      formatter: (params: { name: string; value: number | number[] }) => {
        if (typeof params.value === 'number') {
          const load = params.value
          const status = load > 350
            ? '<span style="color:#ff6b6b;font-weight:700">● 高负荷</span>'
            : load > 200
              ? '<span style="color:#ffb347;font-weight:700">● 中负荷</span>'
              : '<span style="color:#00ff88;font-weight:700">● 正常</span>'
          return `<div style="color:#ffffff;font-weight:700;font-size:14px;margin-bottom:4px">${params.name}</div>`
            + `<div style="color:#d0eeff">用电量：<span style="color:#ffffff;font-weight:700">${load}</span> 万kWh</div>`
            + `<div style="margin-top:2px">状态：${status}</div>`
        }
        if (Array.isArray(params.value) && params.value[2]) {
          const kv = params.value[2] > 70 ? 500 : params.value[2] > 50 ? 220 : 110
          const kvColor = kv === 500 ? '#00ff88' : kv === 220 ? '#00d4ff' : '#9b8ffa'
          return `<div style="color:#ffffff;font-weight:700;font-size:14px;margin-bottom:4px">${params.name}</div>`
            + `<div style="color:#d0eeff">电压等级：<span style="color:${kvColor};font-weight:700">${kv} kV</span></div>`
        }
        return `<span style="color:#ffffff">${params.name}</span>`
      }
    },
    globe: undefined,
    geo3D: {
      map: 'beijing',
      regionHeight: 4,
      roam: true,
      boxWidth: 100,
      boxHeight: 10,
      boxDepth: 80,
      viewControl: {
        distance: 110,
        alpha: 42,
        beta: 2,
        minAlpha: 15,
        maxAlpha: 75,
        autoRotate: false,
        rotateSensitivity: 1,
        zoomSensitivity: 1
      },
      itemStyle: {
        color: 'rgba(0, 25, 70, 0.88)',
        borderWidth: 1,
        borderColor: 'rgba(0, 212, 255, 0.55)'
      },
      emphasis: {
        itemStyle: {
          color: 'rgba(0, 80, 160, 0.95)',
          borderColor: '#00d4ff',
          borderWidth: 2
        },
        label: {
          show: true,
          color: '#ffffff',
          fontSize: 14,
          fontWeight: 'bold',
          textStyle: {
            color: '#ffffff',
            fontSize: 14,
            fontWeight: 'bold',
            backgroundColor: 'rgba(0, 10, 35, 0.88)',
            padding: [4, 10],
            borderRadius: 3
          }
        }
      },
      label: {
        show: false,
        color: '#ffffff',
        textStyle: { color: '#ffffff' }
      },
      light: {
        main: {
          intensity: 1.4,
          shadow: true,
          shadowQuality: 'high',
          alpha: 55,
          beta: 10
        },
        ambient: { intensity: 0.5 },
        ambientCubemap: {
          exposure: 1,
          diffuseIntensity: 0.5
        }
      },
      postEffect: {
        enable: true,
        bloom: { enable: true, bloomIntensity: 0.15 },
        SSAO: { enable: true, quality: 'medium', radius: 2 },
        temporalSuperSampling: { enable: true }
      },
      groundPlane: {
        show: true,
        color: 'rgba(0, 5, 20, 0.95)'
      }
    },
    series: [
      // Bar3D for district loads
      {
        type: 'bar3D',
        coordinateSystem: 'geo3D',
        data: bar3DData,
        bevelSize: 0.3,
        bevelSmoothness: 4,
        shading: 'realistic',
        realisticMaterial: {
          roughness: 0.4,
          metalness: 0.5
        },
        label: { show: false },
        emphasis: {
          label: {
            show: true,
            textStyle: {
              color: '#ffffff',
              fontSize: 13,
              fontWeight: 'bold',
              backgroundColor: 'rgba(0, 10, 35, 0.85)',
              padding: [4, 8],
              borderRadius: 3
            }
          }
        }
      },
      // Lines3D - transmission lines
      {
        type: 'lines3D',
        coordinateSystem: 'geo3D',
        data: lines3DData,
        effect: {
          show: true,
          trailWidth: 2,
          trailLength: 0.25,
          trailOpacity: 1,
          trailColor: '#00d4ff'
        },
        blendMode: 'source-over',
        lineStyle: { width: 1.5, opacity: 0.7 },
        silent: true
      },
      // Scatter3D 500kV
      {
        type: 'scatter3D',
        coordinateSystem: 'geo3D',
        data: scatter500,
        symbol: 'diamond',
        symbolSize: 14,
        itemStyle: { color: '#00ff88', opacity: 1 },
        label: {
          show: true,
          formatter: (p: { name: string }) => p.name,
          textStyle: {
            color: '#ffffff',
            fontSize: 12,
            fontWeight: 'bold',
            backgroundColor: 'rgba(0, 20, 50, 0.82)',
            padding: [3, 6],
            borderRadius: 2,
            borderWidth: 1,
            borderColor: 'rgba(0, 255, 136, 0.5)'
          },
          distance: 8,
          position: 'top'
        }
      },
      // Scatter3D 220kV
      {
        type: 'scatter3D',
        coordinateSystem: 'geo3D',
        data: scatter220,
        symbol: 'circle',
        symbolSize: 9,
        itemStyle: { color: '#00d4ff', opacity: 1 }
      },
      // Scatter3D 110kV
      {
        type: 'scatter3D',
        coordinateSystem: 'geo3D',
        data: scatter110,
        symbol: 'circle',
        symbolSize: 7,
        itemStyle: { color: '#7b68ee', opacity: 1 }
      }
    ]
  }
}

async function loadBeijingMap() {
  try {
    const res = await fetch('/beijing.json')
    const geoJson = await res.json()
    echarts.registerMap('beijing', geoJson)
    mapLoaded = true
  } catch {
    const fallbackGeo = {
      type: 'FeatureCollection',
      features: [
        { type: 'Feature', properties: { name: '朝阳区' }, geometry: { type: 'Polygon', coordinates: [[[116.39, 39.87], [116.62, 39.87], [116.62, 40.08], [116.39, 40.08], [116.39, 39.87]]] } },
        { type: 'Feature', properties: { name: '海淀区' }, geometry: { type: 'Polygon', coordinates: [[[116.17, 39.89], [116.40, 39.89], [116.40, 40.12], [116.17, 40.12], [116.17, 39.89]]] } },
        { type: 'Feature', properties: { name: '丰台区' }, geometry: { type: 'Polygon', coordinates: [[[116.16, 39.75], [116.45, 39.75], [116.45, 39.90], [116.16, 39.90], [116.16, 39.75]]] } },
        { type: 'Feature', properties: { name: '东城区' }, geometry: { type: 'Polygon', coordinates: [[[116.38, 39.88], [116.44, 39.88], [116.44, 39.96], [116.38, 39.96], [116.38, 39.88]]] } },
        { type: 'Feature', properties: { name: '西城区' }, geometry: { type: 'Polygon', coordinates: [[[116.32, 39.88], [116.40, 39.88], [116.40, 39.96], [116.32, 39.96], [116.32, 39.88]]] } },
        { type: 'Feature', properties: { name: '石景山区' }, geometry: { type: 'Polygon', coordinates: [[[116.13, 39.87], [116.22, 39.87], [116.22, 39.96], [116.13, 39.96], [116.13, 39.87]]] } },
        { type: 'Feature', properties: { name: '门头沟区' }, geometry: { type: 'Polygon', coordinates: [[[115.82, 39.82], [116.14, 39.82], [116.14, 40.10], [115.82, 40.10], [115.82, 39.82]]] } },
        { type: 'Feature', properties: { name: '房山区' }, geometry: { type: 'Polygon', coordinates: [[[115.84, 39.54], [116.22, 39.54], [116.22, 39.83], [115.84, 39.83], [115.84, 39.54]]] } },
        { type: 'Feature', properties: { name: '通州区' }, geometry: { type: 'Polygon', coordinates: [[[116.56, 39.72], [116.87, 39.72], [116.87, 40.01], [116.56, 40.01], [116.56, 39.72]]] } },
        { type: 'Feature', properties: { name: '顺义区' }, geometry: { type: 'Polygon', coordinates: [[[116.44, 40.04], [116.86, 40.04], [116.86, 40.32], [116.44, 40.32], [116.44, 40.04]]] } },
        { type: 'Feature', properties: { name: '昌平区' }, geometry: { type: 'Polygon', coordinates: [[[116.02, 40.08], [116.44, 40.08], [116.44, 40.42], [116.02, 40.42], [116.02, 40.08]]] } },
        { type: 'Feature', properties: { name: '大兴区' }, geometry: { type: 'Polygon', coordinates: [[[116.15, 39.44], [116.56, 39.44], [116.56, 39.73], [116.15, 39.73], [116.15, 39.44]]] } },
        { type: 'Feature', properties: { name: '怀柔区' }, geometry: { type: 'Polygon', coordinates: [[[116.32, 40.28], [116.72, 40.28], [116.72, 40.82], [116.32, 40.82], [116.32, 40.28]]] } },
        { type: 'Feature', properties: { name: '平谷区' }, geometry: { type: 'Polygon', coordinates: [[[116.91, 40.02], [117.26, 40.02], [117.26, 40.32], [116.91, 40.32], [116.91, 40.02]]] } },
        { type: 'Feature', properties: { name: '密云区' }, geometry: { type: 'Polygon', coordinates: [[[116.55, 40.30], [117.10, 40.30], [117.10, 40.82], [116.55, 40.82], [116.55, 40.30]]] } },
        { type: 'Feature', properties: { name: '延庆区' }, geometry: { type: 'Polygon', coordinates: [[[115.42, 40.28], [116.10, 40.28], [116.10, 40.82], [115.42, 40.82], [115.42, 40.28]]] } }
      ]
    }
    echarts.registerMap('beijing', fallbackGeo as Parameters<typeof echarts.registerMap>[1])
    mapLoaded = true
  }
}

function switchMode(key: string) {
  activeMode.value = key
}

function handleResize() { chart?.resize() }

watch(activeMode, () => {
  if (chart && mapLoaded) {
    chart.setOption(buildOption(), true)
  }
})

onMounted(async () => {
  if (!chartRef.value) return
  chart = echarts.init(chartRef.value, undefined, { renderer: 'webgl' })

  await loadBeijingMap()

  if (chart && mapLoaded) {
    chart.setOption(buildOption())
  }

  timer = setInterval(() => {
    if (chart && mapLoaded) {
      chart.setOption(buildOption())
    }
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
.map-view {
  height: 100%;
  display: flex;
  flex-direction: column;
  position: relative;
  overflow: hidden;
}

.map-header {
  flex-shrink: 0;
  padding-bottom: 6px;
}

.map-controls {
  margin-left: auto;
  display: flex;
  gap: 4px;
}

.ctrl-btn {
  padding: 3px 12px;
  font-size: 12px;
  border: 1px solid rgba(0, 212, 255, 0.25);
  border-radius: 2px;
  color: var(--text-muted);
  cursor: pointer;
  letter-spacing: 1px;
  transition: all 0.2s;
}

.ctrl-btn:hover,
.ctrl-btn.active {
  background: rgba(0, 212, 255, 0.22);
  border-color: var(--color-primary);
  color: #ffffff;
  box-shadow: 0 0 8px rgba(0, 212, 255, 0.3);
}

.map-stats-row {
  display: flex;
  padding: 6px 14px 4px;
  gap: 0;
  border-top: 1px solid rgba(0, 212, 255, 0.08);
}

.mstat {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  border-right: 1px solid rgba(0, 212, 255, 0.1);
}

.mstat:last-child {
  border-right: none;
}

.mstat-val {
  font-size: 17px;
  font-weight: 700;
}

.mstat-label {
  font-size: 12px;
  color: var(--text-muted);
}

/* Map 3D container */
.map-container-3d {
  flex: 1;
  position: relative;
  overflow: hidden;
  min-height: 0;
}

.map-scan {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: linear-gradient(90deg, transparent, rgba(0, 212, 255, 0.5), transparent);
  animation: scan-line 6s linear infinite;
  z-index: 5;
  pointer-events: none;
}

.map-echarts {
  width: 100%;
  height: 100%;
}

.grid-overlay {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(0, 212, 255, 0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0, 212, 255, 0.03) 1px, transparent 1px);
  background-size: 40px 40px;
  pointer-events: none;
  z-index: 1;
}

/* 3D badge */
.badge-3d {
  position: absolute;
  top: 8px;
  right: 10px;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 1px;
  color: #00d4ff;
  border: 1px solid rgba(0, 212, 255, 0.5);
  border-radius: 3px;
  padding: 1px 6px;
  background: rgba(0, 20, 50, 0.7);
  box-shadow: 0 0 8px rgba(0, 212, 255, 0.3);
  z-index: 6;
  pointer-events: none;
}

/* Legend */
.map-legend {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 16px;
  padding: 6px 12px;
  border-top: 1px solid rgba(0, 212, 255, 0.1);
  background: rgba(1, 8, 25, 0.6);
  flex-shrink: 0;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 12px;
  color: var(--text-secondary);
}

.l-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  flex-shrink: 0;
}

.l-line {
  width: 20px;
  height: 2px;
  border-radius: 1px;
  flex-shrink: 0;
}

@media (max-width: 900px) {
  .map-view {
    height: 420px;
  }

  .map-container-3d {
    min-height: 280px;
  }

  .map-legend {
    flex-wrap: wrap;
    gap: 8px;
  }
}

@media (max-width: 480px) {
  .map-view {
    height: 320px;
  }

  .map-container-3d {
    min-height: 200px;
  }
}
</style>
