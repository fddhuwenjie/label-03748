<template>
  <div class="left-panel">
    <!-- KPI Cards -->
    <div class="kpi-row">
      <div v-for="kpi in kpis" :key="kpi.label" class="kpi-card panel-card">
        <div class="kpi-icon" :style="{ color: kpi.color }">{{ kpi.icon }}</div>
        <div class="kpi-body">
          <div class="kpi-value tech-number" :style="{ color: kpi.color }">
            {{ kpi.value }}<span class="kpi-unit">{{ kpi.unit }}</span>
          </div>
          <div class="kpi-label">{{ kpi.label }}</div>
        </div>
        <div class="kpi-trend" :class="kpi.trend">
          {{ kpi.trend === 'up' ? '▲' : '▼' }} {{ kpi.delta }}
        </div>
      </div>
    </div>

    <!-- Power Load Chart -->
    <PowerLoadChart class="flex-grow" />

    <!-- Generation Pie -->
    <GenerationPie />

    <!-- Substation Status -->
    <SubstationStatus />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import PowerLoadChart from './PowerLoadChart.vue'
import GenerationPie from './GenerationPie.vue'
import SubstationStatus from './SubstationStatus.vue'

interface KPI {
  label: string
  value: string
  unit: string
  icon: string
  color: string
  trend: 'up' | 'down'
  delta: string
}

const kpis = ref<KPI[]>([
  { label: '总发电量', value: '24.86', unit: '亿kWh', icon: '⚡', color: '#00d4ff', trend: 'up', delta: '2.3%' },
  { label: '总用电量', value: '23.41', unit: '亿kWh', icon: '🔋', color: '#00ff88', trend: 'up', delta: '1.8%' }
])

let timer: ReturnType<typeof setInterval>

onMounted(() => {
  timer = setInterval(() => {
    kpis.value[0].value = (24.86 + Math.random() * 0.05).toFixed(2)
    kpis.value[1].value = (23.41 + Math.random() * 0.04).toFixed(2)
  }, 4000)
})

onUnmounted(() => clearInterval(timer))
</script>

<style scoped>
.left-panel {
  display: flex;
  flex-direction: column;
  gap: 8px;
  height: 100%;
  overflow: hidden;
}

.kpi-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  flex-shrink: 0;
}

.kpi-card {
  display: flex;
  align-items: center;
  padding: 8px 10px;
  gap: 8px;
}

.kpi-icon {
  font-size: 20px;
  flex-shrink: 0;
  filter: drop-shadow(0 0 6px currentColor);
}

.kpi-body {
  flex: 1;
  min-width: 0;
}

.kpi-value {
  font-size: 15px;
  font-weight: 700;
  line-height: 1.2;
}

.kpi-unit {
  font-size: 9px;
  font-weight: 400;
  color: var(--text-secondary);
  margin-left: 2px;
}

.kpi-label {
  font-size: 10px;
  color: var(--text-muted);
  margin-top: 2px;
}

.kpi-trend {
  font-size: 10px;
  font-weight: 600;
}

.kpi-trend.up { color: var(--color-accent); }
.kpi-trend.down { color: var(--color-danger); }

.flex-grow {
  flex: 1;
  min-height: 0;
}

@media (max-width: 900px) {
  .left-panel {
    height: auto;
    overflow: visible;
  }

  .flex-grow {
    flex: none;
    min-height: 200px;
    height: 200px;
  }
}
</style>
