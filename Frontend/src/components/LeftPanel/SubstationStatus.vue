<template>
  <div class="panel-card substation-status" role="region" aria-label="关键变电站状态监控">
    <div class="card-title">
      <div class="title-icon" aria-hidden="true"></div>
      <span>关键变电站状态</span>
      <span class="title-unit">实时监控</span>
    </div>
    <div class="station-list" role="list" aria-label="变电站列表">
      <div
        v-for="station in stations"
        :key="station.name"
        class="station-row"
        :class="station.status"
        role="listitem"
        :aria-label="`${station.name}，${station.kv}千伏，负荷率 ${station.load}%，状态：${statusLabel[station.status]}`"
      >
        <div class="station-left">
          <span class="status-dot" :class="station.status" aria-hidden="true"></span>
          <span class="station-name">{{ station.name }}</span>
        </div>
        <div class="station-mid" aria-hidden="true">
          <div class="load-bar-bg">
            <div
              class="load-bar-fill"
              :class="station.status"
              :style="{ width: station.load + '%' }"
            ></div>
          </div>
        </div>
        <div class="station-right" aria-hidden="true">
          <span class="station-load" :class="station.status">{{ station.load }}%</span>
          <span class="station-kv">{{ station.kv }}kV</span>
        </div>
      </div>
    </div>
    <div class="status-summary" role="status"
      :aria-label="`变电站状态汇总：正常 ${onlineCount} 座，预警 ${warnCount} 座，故障 ${offlineCount} 座`">
      <div class="summary-item">
        <span class="sum-dot online" aria-hidden="true"></span>
        <span>正常 <em>{{ onlineCount }}</em></span>
      </div>
      <div class="summary-item">
        <span class="sum-dot warning" aria-hidden="true"></span>
        <span>预警 <em>{{ warnCount }}</em></span>
      </div>
      <div class="summary-item">
        <span class="sum-dot offline" aria-hidden="true"></span>
        <span>故障 <em>{{ offlineCount }}</em></span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

interface Station {
  name: string
  load: number
  kv: number
  status: 'online' | 'warning' | 'offline'
}

const statusLabel: Record<string, string> = {
  online: '正常',
  warning: '预警',
  offline: '故障'
}

const STATION_NAMES = [
  '昌平变电站', '顺义变电站', '通州变电站',
  '大兴变电站', '房山变电站', '密云变电站',
  '延庆变电站', '平谷变电站'
]

const KV_LEVELS = [500, 220, 110, 220, 110, 220, 110, 110]

function generateStations(): Station[] {
  return STATION_NAMES.map((name, i) => {
    const load = Math.round(55 + Math.random() * 40)
    const status: Station['status'] =
      load > 90 ? 'offline' : load > 80 ? 'warning' : 'online'
    return { name, load, kv: KV_LEVELS[i], status }
  })
}

const stations = ref<Station[]>(generateStations())
const onlineCount = computed(() => stations.value.filter(s => s.status === 'online').length)
const warnCount = computed(() => stations.value.filter(s => s.status === 'warning').length)
const offlineCount = computed(() => stations.value.filter(s => s.status === 'offline').length)

let timer: ReturnType<typeof setInterval>

onMounted(() => {
  timer = setInterval(() => {
    stations.value = generateStations()
  }, 6000)
})

onUnmounted(() => clearInterval(timer))
</script>

<style scoped>
.substation-status {
  flex: 0 0 auto;
}

.station-list {
  padding: 6px 12px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.station-row {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 4px 6px;
  border-radius: 3px;
  background: rgba(0, 50, 80, 0.15);
  transition: background 0.3s;
}

.station-row:hover {
  background: rgba(0, 100, 150, 0.2);
}

.station-left {
  display: flex;
  align-items: center;
  gap: 6px;
  width: 90px;
}

.station-name {
  font-size: 12px;
  color: var(--text-secondary);
  white-space: nowrap;
}

.station-mid {
  flex: 1;
}

.load-bar-bg {
  height: 4px;
  background: rgba(0, 50, 80, 0.4);
  border-radius: 2px;
  overflow: hidden;
}

.load-bar-fill {
  height: 100%;
  border-radius: 2px;
  transition: width 0.8s ease;
}

.load-bar-fill.online { background: linear-gradient(90deg, #00a0c0, #00d4ff); }
.load-bar-fill.warning { background: linear-gradient(90deg, #cc7700, #ff9500); }
.load-bar-fill.offline { background: linear-gradient(90deg, #cc0000, #ff4444); }

.station-right {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 1px;
}

.station-load {
  font-size: 12px;
  font-family: 'Courier New', monospace;
  font-weight: 700;
}

.station-load.online { color: var(--color-primary); }
.station-load.warning { color: var(--color-warning); }
.station-load.offline { color: var(--color-danger); }

.station-kv {
  font-size: 12px;
  color: var(--text-muted);
}

.status-summary {
  display: flex;
  justify-content: space-around;
  padding: 6px 12px 8px;
  border-top: 1px solid rgba(0, 212, 255, 0.08);
  font-size: 12px;
  color: var(--text-secondary);
}

.summary-item {
  display: flex;
  align-items: center;
  gap: 5px;
}

.summary-item em {
  font-style: normal;
  font-weight: 700;
  font-family: 'Courier New', monospace;
  color: var(--color-primary);
}

.sum-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}

.sum-dot.online { background: var(--color-accent); }
.sum-dot.warning { background: var(--color-warning); }
.sum-dot.offline { background: var(--color-danger); }
</style>
