<template>
  <div class="panel-card alarm-list">
    <div class="card-title">
      <div class="title-icon"></div>
      <span>实时告警信息</span>
      <div class="alarm-badge" :class="{ blink: alarms.length > 0 }">{{ alarms.length }}</div>
    </div>
    <div class="alarm-scroll-wrap" ref="scrollWrap">
      <div class="alarm-scroll-inner" ref="scrollInner">
        <div
          v-for="alarm in displayAlarms"
          :key="alarm.id"
          class="alarm-row"
          :class="alarm.level"
        >
          <span class="alarm-level-tag" :class="alarm.level">{{ levelLabel[alarm.level] }}</span>
          <div class="alarm-content">
            <div class="alarm-msg">{{ alarm.message }}</div>
            <div class="alarm-meta">{{ alarm.station }} · {{ alarm.time }}</div>
          </div>
          <span class="alarm-status" :class="alarm.handled ? 'handled' : 'pending'">
            {{ alarm.handled ? '已处理' : '处理中' }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

interface Alarm {
  id: number
  level: 'critical' | 'warning' | 'info'
  message: string
  station: string
  time: string
  handled: boolean
}

const levelLabel: Record<string, string> = {
  critical: '紧急',
  warning: '预警',
  info: '通知'
}

const MESSAGES = {
  critical: [
    '线路过载，负荷超过额定容量95%',
    '变压器温度异常，已超过85°C',
    '110kV线路跳闸，需立即处置',
    '母线差动保护动作'
  ],
  warning: [
    '功率因数低于0.9，请检查无功补偿',
    '电压偏差超过允许范围±7%',
    '线路负荷率超过额定值80%',
    '主变压器油温偏高，建议检查'
  ],
  info: [
    '设备定期巡检提醒',
    '电容器组投入运行',
    '备用线路切换完成',
    '继电保护装置定值核查完成'
  ]
}

const STATIONS = [
  '昌平站', '顺义站', '通州站', '大兴站',
  '房山站', '密云站', '延庆站', '石景山站',
  '朝阳站', '海淀站'
]

let nextId = 1

function randomAlarm(): Alarm {
  const levels: Alarm['level'][] = ['critical', 'warning', 'warning', 'info', 'info', 'info']
  const level = levels[Math.floor(Math.random() * levels.length)]
  const msgs = MESSAGES[level]
  const msg = msgs[Math.floor(Math.random() * msgs.length)]
  const station = STATIONS[Math.floor(Math.random() * STATIONS.length)]
  const now = new Date()
  const pad = (n: number) => String(n).padStart(2, '0')
  const time = `${pad(now.getHours())}:${pad(now.getMinutes())}:${pad(now.getSeconds())}`
  return {
    id: nextId++,
    level,
    message: msg,
    station,
    time,
    handled: Math.random() > 0.4
  }
}

const alarms = ref<Alarm[]>([])

for (let i = 0; i < 8; i++) {
  alarms.value.push(randomAlarm())
}

const displayAlarms = computed(() => [...alarms.value].reverse())

const scrollWrap = ref<HTMLElement>()
let scrollTimer: ReturnType<typeof setInterval>
let alarmTimer: ReturnType<typeof setInterval>
let scrollPos = 0

onMounted(() => {
  alarmTimer = setInterval(() => {
    alarms.value.push(randomAlarm())
    if (alarms.value.length > 20) alarms.value.shift()
  }, 4000)

  scrollTimer = setInterval(() => {
    if (!scrollWrap.value) return
    const el = scrollWrap.value
    scrollPos += 1
    if (scrollPos >= el.scrollHeight - el.clientHeight) {
      scrollPos = 0
    }
    el.scrollTop = scrollPos
  }, 60)
})

onUnmounted(() => {
  clearInterval(scrollTimer)
  clearInterval(alarmTimer)
})
</script>

<style scoped>
.alarm-list {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.alarm-badge {
  margin-left: auto;
  background: var(--color-danger);
  color: #fff;
  font-size: 10px;
  font-weight: 700;
  padding: 1px 6px;
  border-radius: 8px;
  min-width: 18px;
  text-align: center;
  box-shadow: 0 0 6px var(--color-danger);
}

.alarm-badge.blink {
  animation: blink 1.5s ease-in-out infinite;
}

.alarm-scroll-wrap {
  flex: 1;
  overflow: hidden;
  min-height: 0;
}

.alarm-scroll-inner {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding: 6px 10px;
}

.alarm-row {
  display: flex;
  align-items: flex-start;
  gap: 7px;
  padding: 6px 8px;
  border-radius: 3px;
  background: rgba(0, 30, 60, 0.3);
  border-left: 2px solid transparent;
  transition: all 0.3s;
}

.alarm-row.critical {
  border-left-color: var(--color-danger);
  background: rgba(80, 0, 0, 0.2);
}

.alarm-row.warning {
  border-left-color: var(--color-warning);
  background: rgba(60, 30, 0, 0.2);
}

.alarm-row.info {
  border-left-color: rgba(0, 212, 255, 0.4);
}

.alarm-level-tag {
  flex-shrink: 0;
  font-size: 9px;
  font-weight: 700;
  padding: 2px 5px;
  border-radius: 2px;
  letter-spacing: 0.5px;
  margin-top: 1px;
}

.alarm-level-tag.critical {
  background: rgba(255, 68, 68, 0.25);
  color: var(--color-danger);
  border: 1px solid rgba(255, 68, 68, 0.4);
}

.alarm-level-tag.warning {
  background: rgba(255, 149, 0, 0.2);
  color: var(--color-warning);
  border: 1px solid rgba(255, 149, 0, 0.35);
}

.alarm-level-tag.info {
  background: rgba(0, 212, 255, 0.1);
  color: var(--color-primary);
  border: 1px solid rgba(0, 212, 255, 0.25);
}

.alarm-content {
  flex: 1;
  min-width: 0;
}

.alarm-msg {
  font-size: 11px;
  color: var(--text-primary);
  line-height: 1.4;
}

.alarm-meta {
  font-size: 10px;
  color: var(--text-muted);
  margin-top: 2px;
}

.alarm-status {
  flex-shrink: 0;
  font-size: 9px;
  padding: 2px 5px;
  border-radius: 2px;
  margin-top: 1px;
}

.alarm-status.handled {
  color: var(--color-accent);
  background: rgba(0, 255, 136, 0.1);
  border: 1px solid rgba(0, 255, 136, 0.25);
}

.alarm-status.pending {
  color: var(--color-warning);
  background: rgba(255, 149, 0, 0.1);
  border: 1px solid rgba(255, 149, 0, 0.25);
}
</style>
