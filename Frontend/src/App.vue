<template>
  <div class="dashboard">
    <!-- Scan line effect -->
    <div class="scan-line"></div>

    <!-- Header -->
    <header class="dashboard-header" role="banner" aria-label="电网驾驶舱头部信息">
      <div class="header-left" aria-label="时间与系统指标">
        <div class="time-display">
          <div class="current-time tech-number" aria-live="polite" aria-label="当前时间">{{ currentTime }}</div>
          <div class="current-date" aria-live="polite">{{ currentDate }}</div>
        </div>
        <div class="header-stats" role="list" aria-label="系统运行指标">
          <div class="hstat" role="listitem">
            <span class="hstat-label">系统频率</span>
            <span class="hstat-value tech-number" aria-label="系统频率 50.00 赫兹">50.00<em aria-hidden="true">Hz</em></span>
          </div>
          <div class="hstat" role="listitem">
            <span class="hstat-label">运行天数</span>
            <span class="hstat-value tech-number" :aria-label="`运行天数 ${runDays} 天`">{{ runDays }}<em aria-hidden="true">天</em></span>
          </div>
        </div>
      </div>

      <div class="header-center">
        <div class="title-decoration-row" aria-hidden="true">
          <div class="deco-line"></div>
          <div class="deco-diamond"></div>
          <div class="deco-line"></div>
        </div>
        <h1 class="main-title">
          <span class="title-icon-left" aria-hidden="true">⚡</span>
          中国电网驾驶舱
          <span class="title-icon-right" aria-hidden="true">⚡</span>
        </h1>
        <div class="title-en" aria-hidden="true">CHINA POWER GRID INTELLIGENT COCKPIT SYSTEM</div>
        <div class="title-decoration-row bottom" aria-hidden="true">
          <div class="deco-line"></div>
          <div class="deco-dots">
            <span v-for="i in 5" :key="i"></span>
          </div>
          <div class="deco-line"></div>
        </div>
      </div>

      <div class="header-right" aria-label="系统状态与负荷">
        <div class="sys-status" role="status" aria-label="系统运行状态：正常">
          <span class="status-dot active" aria-hidden="true"></span>
          <span class="status-label">系统运行正常</span>
        </div>
        <div class="header-kpis" role="list" aria-label="关键负荷指标">
          <div class="hstat" role="listitem">
            <span class="hstat-label">总负荷</span>
            <span class="hstat-value tech-number" :aria-label="`总负荷 ${totalLoad} 万千瓦`">{{ totalLoad }}<em aria-hidden="true">万kW</em></span>
          </div>
          <div class="hstat" role="listitem">
            <span class="hstat-label">在线站点</span>
            <span class="hstat-value tech-number" :aria-label="`在线站点 ${onlineCount} 座`">{{ onlineCount }}<em aria-hidden="true">座</em></span>
          </div>
        </div>
        <div class="weather-row" aria-label="天气信息">
          <span class="weather-text">北京 多云 12°C &nbsp;湿度 45%</span>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="dashboard-main" role="main" aria-label="电网监控主界面">
      <LeftPanel class="panel-section" />
      <MapView class="panel-section map-section" />
      <RightPanel class="panel-section" />
    </main>

    <!-- Footer -->
    <footer class="dashboard-footer" role="contentinfo" aria-label="今日电网数据摘要">
      <div class="footer-left">
        <span>国家电网有限公司 &nbsp;STATE GRID CORPORATION OF CHINA</span>
      </div>
      <div class="footer-center" role="list" aria-label="今日电力统计">
        <span class="footer-stat" role="listitem">今日发电: <em class="tech-number" :aria-label="`今日发电 ${todayGen} 亿千瓦时`">{{ todayGen }}</em><span aria-hidden="true"> 亿kWh</span></span>
        <span class="footer-sep" aria-hidden="true">◆</span>
        <span class="footer-stat" role="listitem">今日用电: <em class="tech-number" :aria-label="`今日用电 ${todayUsed} 亿千瓦时`">{{ todayUsed }}</em><span aria-hidden="true"> 亿kWh</span></span>
        <span class="footer-sep" aria-hidden="true">◆</span>
        <span class="footer-stat" role="listitem">电网告警: <em class="tech-number warn" :aria-label="`电网告警 ${alarmCount} 条`">{{ alarmCount }}</em><span aria-hidden="true"> 条</span></span>
      </div>
      <div class="footer-right">
        <span>数据更新: <time :datetime="updateTime">{{ updateTime }}</time></span>
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
  font-size: 12px;
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
  font-size: 12px;
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
  font-size: 12px;
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
  font-size: 11px;
  letter-spacing: 3px;
  /* 不透明度从 0.55 提升至 0.75，改善对比度 */
  color: rgba(0, 212, 255, 0.75);
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
  font-size: 12px;
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
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  background: rgba(1, 8, 25, 0.95);
  border-top: 1px solid rgba(0, 212, 255, 0.15);
  font-size: 12px;
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
  /* 不透明度从 0.3 提升至 0.5，字号从 8px 提升至 10px */
  color: rgba(0, 212, 255, 0.5);
  font-size: 10px;
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
    font-size: 10px;
    letter-spacing: 1px;
  }

  .current-time {
    font-size: 18px;
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
    font-size: 16px;
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

/* ========== Large Screen Adaptation ========== */
@media (min-width: 1600px) {
  .dashboard-header {
    height: 108px;
    padding: 0 40px;
  }

  .main-title {
    font-size: 34px;
  }

  .current-time {
    font-size: 30px;
  }

  .hstat-label {
    font-size: 13px;
  }

  .hstat-value {
    font-size: 16px;
  }

  .hstat-value em {
    font-size: 13px;
  }

  .dashboard-footer {
    height: 36px;
    font-size: 13px;
  }

  .dashboard-main {
    gap: 12px;
    padding: 12px;
  }
}

@media (min-width: 2560px) {
  .dashboard-header {
    height: 130px;
    padding: 0 60px;
  }

  .main-title {
    font-size: 42px;
  }

  .current-time {
    font-size: 38px;
  }

  .hstat-label {
    font-size: 15px;
  }

  .hstat-value {
    font-size: 20px;
  }

  .hstat-value em {
    font-size: 15px;
  }

  .dashboard-footer {
    height: 44px;
    font-size: 15px;
    padding: 0 40px;
  }

  .dashboard-main {
    gap: 16px;
    padding: 16px;
  }
}
</style>
