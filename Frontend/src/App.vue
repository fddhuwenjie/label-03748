<template>
  <div class="dashboard">
    <!-- Scan line effect -->
    <div class="scan-line"></div>

    <!-- Header -->
    <header class="dashboard-header">
      <div class="header-left">
        <div class="time-display">
          <div class="current-time tech-number">{{ currentTime }}</div>
          <div class="current-date">{{ currentDate }}</div>
        </div>
        <div class="header-stats">
          <div class="hstat">
            <span class="hstat-label">系统频率</span>
            <span class="hstat-value tech-number">50.00<em>Hz</em></span>
          </div>
          <div class="hstat">
            <span class="hstat-label">运行天数</span>
            <span class="hstat-value tech-number">{{ runDays }}<em>天</em></span>
          </div>
        </div>
      </div>

      <div class="header-center">
        <div class="title-decoration-row">
          <div class="deco-line"></div>
          <div class="deco-diamond"></div>
          <div class="deco-line"></div>
        </div>
        <h1 class="main-title">
          <span class="title-icon-left">⚡</span>
          中国电网驾驶舱
          <span class="title-icon-right">⚡</span>
        </h1>
        <div class="title-en">CHINA POWER GRID INTELLIGENT COCKPIT SYSTEM</div>
        <div class="title-decoration-row bottom">
          <div class="deco-line"></div>
          <div class="deco-dots">
            <span v-for="i in 5" :key="i"></span>
          </div>
          <div class="deco-line"></div>
        </div>
      </div>

      <div class="header-right">
        <div class="sys-status">
          <span class="status-dot active"></span>
          <span class="status-label">系统运行正常</span>
        </div>
        <div class="header-kpis">
          <div class="hstat">
            <span class="hstat-label">总负荷</span>
            <span class="hstat-value tech-number">{{ totalLoad }}<em>万kW</em></span>
          </div>
          <div class="hstat">
            <span class="hstat-label">在线站点</span>
            <span class="hstat-value tech-number">{{ onlineCount }}<em>座</em></span>
          </div>
        </div>
        <div class="weather-row">
          <span class="weather-text">北京 多云 12°C  湿度 45%</span>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="dashboard-main">
      <LeftPanel class="panel-section" />
      <MapView class="panel-section map-section" />
      <RightPanel class="panel-section" />
    </main>

    <!-- Footer -->
    <footer class="dashboard-footer">
      <div class="footer-left">
        <span>国家电网有限公司 &nbsp;STATE GRID CORPORATION OF CHINA</span>
      </div>
      <div class="footer-center">
        <span class="footer-stat">今日发电: <em class="tech-number">{{ todayGen }}</em> 亿kWh</span>
        <span class="footer-sep">◆</span>
        <span class="footer-stat">今日用电: <em class="tech-number">{{ todayUsed }}</em> 亿kWh</span>
        <span class="footer-sep">◆</span>
        <span class="footer-stat">电网告警: <em class="tech-number warn">{{ alarmCount }}</em> 条</span>
      </div>
      <div class="footer-right">
        <span>数据更新: {{ updateTime }}</span>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import LeftPanel from './components/LeftPanel/index.vue'
import RightPanel from './components/RightPanel/index.vue'
import MapView from './components/MapView/index.vue'

const currentTime = ref('')
const currentDate = ref('')
const updateTime = ref('')
const runDays = ref(1286)
const totalLoad = ref('8,742')
const onlineCount = ref(312)
const todayGen = ref('24.86')
const todayUsed = ref('23.41')
const alarmCount = ref(7)

let timer: ReturnType<typeof setInterval>

function updateDateTime() {
  const now = new Date()
  const pad = (n: number) => String(n).padStart(2, '0')
  currentTime.value = `${pad(now.getHours())}:${pad(now.getMinutes())}:${pad(now.getSeconds())}`
  currentDate.value = `${now.getFullYear()}年${pad(now.getMonth() + 1)}月${pad(now.getDate())}日 星期${'日一二三四五六'[now.getDay()]}`
  updateTime.value = `${pad(now.getHours())}:${pad(now.getMinutes())}:${pad(now.getSeconds())}`
}

function updateStats() {
  const base = 8742
  const delta = Math.round((Math.random() - 0.5) * 30)
  totalLoad.value = (base + delta).toLocaleString()
  const genBase = 24.86
  todayGen.value = (genBase + Math.random() * 0.02).toFixed(2)
  todayUsed.value = (23.41 + Math.random() * 0.02).toFixed(2)
}

onMounted(() => {
  updateDateTime()
  timer = setInterval(() => {
    updateDateTime()
    if (Math.random() < 0.1) updateStats()
  }, 1000)
})

onUnmounted(() => clearInterval(timer))
</script>

<style scoped>
.dashboard {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #050a1a;
  overflow: hidden;
  position: relative;
}

.scan-line {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(0, 212, 255, 0.4), transparent);
  animation: scan-line 8s linear infinite;
  z-index: 100;
  pointer-events: none;
}

/* ========== Header ========== */
.dashboard-header {
  height: 96px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 28px;
  background: rgba(2, 10, 30, 0.95);
  border-bottom: 1px solid rgba(0, 212, 255, 0.2);
  position: relative;
  flex-shrink: 0;
  z-index: 10;
}

.dashboard-header::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 60%;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(0, 212, 255, 0.6), transparent);
}

.header-left, .header-right {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.header-right {
  align-items: flex-end;
}

.time-display {
  line-height: 1.2;
}

.current-time {
  font-size: 26px;
  font-weight: 700;
  letter-spacing: 3px;
}

.current-date {
  font-size: 11px;
  color: var(--text-secondary);
  letter-spacing: 1px;
}

.header-stats, .header-kpis {
  display: flex;
  gap: 16px;
  margin-top: 2px;
}

.hstat {
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.hstat-label {
  font-size: 10px;
  color: var(--text-muted);
  letter-spacing: 0.5px;
}

.hstat-value {
  font-size: 14px;
  font-weight: 700;
  color: var(--color-primary);
}

.hstat-value em {
  font-style: normal;
  font-size: 10px;
  color: var(--text-secondary);
  margin-left: 2px;
}

/* Header Center */
.header-center {
  flex: 0 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.title-decoration-row {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
}

.deco-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(0, 212, 255, 0.5));
}

.title-decoration-row .deco-line:last-child {
  background: linear-gradient(270deg, transparent, rgba(0, 212, 255, 0.5));
}

.deco-diamond {
  width: 6px;
  height: 6px;
  background: var(--color-primary);
  transform: rotate(45deg);
  box-shadow: 0 0 8px var(--color-primary);
}

.main-title {
  font-size: 30px;
  font-weight: 700;
  letter-spacing: 4px;
  color: #fff;
  text-shadow: 0 0 20px rgba(0, 212, 255, 0.6), 0 0 40px rgba(0, 212, 255, 0.3);
  white-space: nowrap;
}

.title-icon-left, .title-icon-right {
  font-size: 20px;
  animation: pulse 2s ease-in-out infinite;
}

.title-en {
  font-size: 10px;
  letter-spacing: 3px;
  color: rgba(0, 212, 255, 0.55);
  white-space: nowrap;
}

.deco-dots {
  display: flex;
  gap: 4px;
}

.deco-dots span {
  display: inline-block;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: var(--color-primary);
  opacity: 0.5;
}

.deco-dots span:nth-child(3) { opacity: 1; box-shadow: 0 0 6px var(--color-primary); }

/* Header Right */
.sys-status {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: var(--color-accent);
}

.status-label {
  letter-spacing: 0.5px;
}

.weather-row {
  margin-top: 2px;
}

.weather-text {
  font-size: 11px;
  color: var(--text-muted);
}

.hstat-value.warn {
  color: var(--color-warning);
}

/* ========== Main ========== */
.dashboard-main {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 8px;
  padding: 8px;
  overflow: hidden;
  min-height: 0;
}

.panel-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
  overflow: hidden;
  min-height: 0;
}

.map-section {
  position: relative;
}

/* ========== Footer ========== */
.dashboard-footer {
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  background: rgba(1, 8, 25, 0.95);
  border-top: 1px solid rgba(0, 212, 255, 0.15);
  font-size: 11px;
  color: var(--text-muted);
  flex-shrink: 0;
}

.footer-center {
  display: flex;
  align-items: center;
  gap: 12px;
}

.footer-stat em {
  font-style: normal;
  font-family: 'Courier New', monospace;
  color: var(--color-primary);
}

.footer-stat em.warn {
  color: var(--color-warning);
}

.footer-sep {
  color: rgba(0, 212, 255, 0.3);
  font-size: 8px;
}

/* ========== Responsive: Small Screen ========== */
@media (max-width: 900px) {
  .dashboard {
    height: auto;
    min-height: 100vh;
    overflow-y: auto;
    overflow-x: hidden;
  }

  /* Header: stack into two rows */
  .dashboard-header {
    height: auto;
    flex-direction: column;
    align-items: center;
    padding: 10px 14px;
    gap: 8px;
  }

  .header-left,
  .header-right {
    flex: none;
    width: 100%;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
  }

  .header-right {
    align-items: center;
  }

  .header-center {
    order: -1;
    width: 100%;
  }

  .main-title {
    font-size: 18px;
    letter-spacing: 2px;
  }

  .title-en {
    font-size: 8px;
    letter-spacing: 1px;
  }

  .current-time {
    font-size: 16px;
  }

  /* Main: single column */
  .dashboard-main {
    grid-template-columns: 1fr;
    overflow: visible;
  }

  .panel-section {
    overflow: visible;
    min-height: unset;
  }

  /* Footer: wrap text */
  .dashboard-footer {
    height: auto;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    padding: 8px 14px;
    text-align: center;
  }

  .footer-center {
    flex-wrap: wrap;
    justify-content: center;
    gap: 8px;
  }
}

@media (max-width: 480px) {
  .main-title {
    font-size: 15px;
    letter-spacing: 1px;
  }

  .title-en {
    display: none;
  }

  .header-stats,
  .header-kpis {
    gap: 10px;
  }

  .weather-row {
    display: none;
  }
}
</style>
