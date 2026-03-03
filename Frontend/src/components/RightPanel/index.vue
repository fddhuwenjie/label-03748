<template>
  <div class="right-panel">
    <!-- KPI Cards -->
    <el-row :gutter="8" class="kpi-row" role="list" aria-label="电网质量指标">
      <el-col :span="12" v-for="kpi in kpis" :key="kpi.label">
        <div class="kpi-card panel-card" role="listitem"
          :aria-label="`${kpi.label}：${kpi.value} ${kpi.unit}，占比 ${kpi.pct}%`">
          <div class="kpi-left">
            <div class="kpi-value tech-number" :style="{ color: kpi.color }" aria-hidden="true">
              {{ kpi.value }}<span class="kpi-unit">{{ kpi.unit }}</span>
            </div>
            <div class="kpi-label">{{ kpi.label }}</div>
          </div>
          <div class="kpi-ring" :style="{ '--pct': kpi.pct + '%', '--color': kpi.color }"
            role="img" :aria-label="`${kpi.pct}%`">
            <span class="ring-val" aria-hidden="true">{{ kpi.pct }}%</span>
          </div>
        </div>
      </el-col>
    </el-row>

    <!-- Alarm List -->
    <AlarmList class="flex-grow" />

    <!-- Line Status Chart -->
    <LineStatusChart />

    <!-- Energy Consumption Chart -->
    <EnergyConsumption />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { ElRow, ElCol } from 'element-plus'
import AlarmList from './AlarmList.vue'
import LineStatusChart from './LineStatusChart.vue'
import EnergyConsumption from './EnergyConsumption.vue'

interface KPI {
  label: string
  value: string
  unit: string
  color: string
  pct: number
}

const kpis = ref<KPI[]>([
  { label: '电网可靠率', value: '99.98', unit: '%', color: '#00ff88', pct: 99 },
  { label: '综合线损率', value: '3.42', unit: '%', color: '#ff9500', pct: 34 }
])

let timer: ReturnType<typeof setInterval>

onMounted(() => {
  timer = setInterval(() => {
    kpis.value[0].value = (99.97 + Math.random() * 0.02).toFixed(2)
    kpis.value[1].value = (3.38 + Math.random() * 0.08).toFixed(2)
  }, 5000)
})

onUnmounted(() => clearInterval(timer))
</script>

<style scoped>
.right-panel {
  display: flex;
  flex-direction: column;
  gap: 8px;
  height: 100%;
  overflow: hidden;
}

.kpi-row {
  flex-shrink: 0;
  margin-bottom: 0 !important;
}

.kpi-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 10px;
}

.kpi-value {
  font-size: 15px;
  font-weight: 700;
  line-height: 1.2;
}

.kpi-unit {
  font-size: 12px;
  font-weight: 400;
  color: var(--text-secondary);
  margin-left: 1px;
}

.kpi-label {
  font-size: 12px;
  color: var(--text-muted);
  margin-top: 3px;
}

/* Ring progress */
.kpi-ring {
  position: relative;
  width: 38px;
  height: 38px;
  border-radius: 50%;
  background: conic-gradient(var(--color) var(--pct), rgba(0,50,80,0.3) var(--pct));
  display: flex;
  align-items: center;
  justify-content: center;
}

.kpi-ring::before {
  content: '';
  position: absolute;
  inset: 5px;
  background: var(--bg-panel-dark);
  border-radius: 50%;
}

.ring-val {
  position: relative;
  font-size: 11px;
  font-family: 'Courier New', monospace;
  color: var(--text-secondary);
  z-index: 1;
}

.flex-grow {
  flex: 1;
  min-height: 0;
}

@media (max-width: 900px) {
  .right-panel {
    height: auto;
    overflow: visible;
  }

  .flex-grow {
    flex: none;
    min-height: 200px;
    height: 200px;
  }
}

@media (min-width: 1600px) {
  .kpi-value {
    font-size: 18px;
  }

  .kpi-unit,
  .kpi-label {
    font-size: 13px;
  }

  .kpi-ring {
    width: 46px;
    height: 46px;
  }

  .ring-val {
    font-size: 12px;
  }
}

@media (min-width: 2560px) {
  .kpi-value {
    font-size: 22px;
  }

  .kpi-unit,
  .kpi-label {
    font-size: 15px;
  }

  .kpi-ring {
    width: 56px;
    height: 56px;
  }

  .ring-val {
    font-size: 14px;
  }
}
</style>
