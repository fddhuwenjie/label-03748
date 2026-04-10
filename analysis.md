# Vue3电力驾驶舱大屏项目源码分析

## 一、MapView组件3D北京地图实现分析

### 1.1 使用的库

3D北京地图使用 **ECharts + ECharts-GL** 实现：

```typescript
import * as echarts from 'echarts'
import 'echarts-gl'
```

- **echarts**: 基础图表库
- **echarts-gl**: ECharts的3D扩展库，提供WebGL渲染能力，支持3D地图、散点、柱状图等

### 1.2 地图数据来源

地图数据通过加载北京GeoJSON数据实现：

```typescript
async function loadBeijingMap() {
  try {
    const res = await fetch('/beijing.json')
    const geoJson = await res.json()
    echarts.registerMap('beijing', geoJson)
  } catch {
    // 降级方案：使用内置的简化版北京区域数据
    const fallbackGeo = { ... }
    echarts.registerMap('beijing', fallbackGeo)
  }
}
```

### 1.3 光照配置

位于 `geo3D.light` 配置项：

```javascript
light: {
  main: {
    intensity: 1.4,        // 主光源强度
    shadow: true,          // 开启阴影
    shadowQuality: 'high', // 阴影质量
    alpha: 55,             // 主光源垂直角度
    beta: 10               // 主光源水平角度
  },
  ambient: { intensity: 0.5 },  // 环境光强度
  ambientCubemap: {
    exposure: 1,           // 环境贴图曝光
    diffuseIntensity: 0.5  // 漫反射强度
  }
}
```

### 1.4 材质配置

**地图区域材质**（geo3D.itemStyle）：

```javascript
itemStyle: {
  color: 'rgba(0, 25, 70, 0.88)',        // 基础颜色
  borderWidth: 1,                         // 边框宽度
  borderColor: 'rgba(0, 212, 255, 0.55)'  // 边框颜色（科技蓝）
}
```

**3D柱状图材质**（bar3D.realisticMaterial）：

```javascript
shading: 'realistic',  // 真实感渲染
realisticMaterial: {
  roughness: 0.4,      // 粗糙度
  metalness: 0.5       // 金属度
}
```

### 1.5 相机参数配置

位于 `geo3D.viewControl`：

```javascript
viewControl: {
  distance: 110,         // 相机距离
  alpha: 42,             // 俯仰角（垂直旋转角度）
  beta: 2,               // 水平旋转角度
  minAlpha: 15,          // 最小俯仰角
  maxAlpha: 75,          // 最大俯仰角
  autoRotate: false,     // 自动旋转
  rotateSensitivity: 1,  // 旋转灵敏度
  zoomSensitivity: 1     // 缩放灵敏度
}
```

### 1.6 3D场景其他配置

**后处理效果**（postEffect）：

```javascript
postEffect: {
  enable: true,
  bloom: { enable: true, bloomIntensity: 0.15 },  // 泛光效果
  SSAO: { enable: true, quality: 'medium', radius: 2 },  // 环境光遮蔽
  temporalSuperSampling: { enable: true }  // 时间超采样抗锯齿
}
```

**地面平面**：

```javascript
groundPlane: {
  show: true,
  color: 'rgba(0, 5, 20, 0.95)'
}
```

### 1.7 3D图表类型

MapView组件中使用了以下ECharts-GL 3D系列类型：

| 系列类型 | 用途 | 说明 |
|---------|------|------|
| `bar3D` | 各区用电量柱状图 | 展示16个区的用电量高度 |
| `lines3D` | 输电线路 | 带流动效果的3D线条 |
| `scatter3D` | 变电站位置 | 500kV/220kV/110kV不同等级 |

---

## 二、左右面板6个图表组件分析

### 2.1 左面板组件（LeftPanel）

#### 1. PowerLoadChart - 实时电网负荷曲线

- **图表类型**: `line`（折线图）+ `scatter`（散点图）
- **数据类型**: **静态Mock数据** + 随机波动

```typescript
const BASE_LOADS = [
  420, 385, 355, 330, 318, 336, 495, 658,
  782, 856, 872, 845, 808, 825, 855, 874,
  892, 928, 882, 805, 724, 642, 562, 490
]

function getLoadData() {
  return BASE_LOADS.map(v => v + Math.round((Math.random() - 0.5) * 25))
}
```

- **更新频率**: 每5秒更新一次（`setInterval(..., 5000)`）
- **特性**: 带渐变填充区域的平滑曲线，当前时间点高亮显示

#### 2. GenerationPie - 发电能源结构

- **图表类型**: `pie`（饼图/环形图）
- **数据类型**: **静态Mock数据** + 随机波动

```typescript
const SOURCES = [
  { name: '火力发电', baseVal: 9.2 },
  { name: '水力发电', baseVal: 5.8 },
  { name: '核能发电', baseVal: 3.4 },
  { name: '风力发电', baseVal: 4.1 },
  { name: '光伏发电', baseVal: 2.9 },
  { name: '其他新能源', baseVal: 1.2 }
]
```

- **更新频率**: 每8秒更新一次
- **特性**: 环形饼图（radius: ['38%', '68%']），自定义图例列表

#### 3. SubstationStatus - 关键变电站状态

- **图表类型**: **非ECharts图表**，纯Vue组件实现
- **数据类型**: **静态Mock数据** + 随机波动

```typescript
const STATION_NAMES = [
  '昌平变电站', '顺义变电站', '通州变电站',
  '大兴变电站', '房山变电站', '密云变电站',
  '延庆变电站', '平谷变电站'
]

function generateStations(): Station[] {
  return STATION_NAMES.map((name, i) => {
    const load = Math.round(55 + Math.random() * 40)
    const status: Station['status'] =
      load > 90 ? 'offline' : load > 80 ? 'warning' : 'online'
    return { name, load, kv: KV_LEVELS[i], status }
  })
}
```

- **更新频率**: 每6秒更新一次
- **特性**: 列表形式展示，带负荷率进度条和状态指示

### 2.2 右面板组件（RightPanel）

#### 4. EnergyConsumption - 各区用电量统计

- **图表类型**: `bar`（柱状图）
- **数据类型**: **静态Mock数据** + 随机波动

```typescript
const DISTRICTS = ['朝阳', '海淀', '丰台', '西城', '东城', '石景山', '顺义', '大兴']
const BASE_VALUES = [420, 388, 310, 195, 172, 135, 285, 265]

function getConsumption() {
  return BASE_VALUES.map(v => Math.round(v + (Math.random() - 0.5) * 30))
}
```

- **更新频率**: 每6秒更新一次
- **特性**: 渐变色柱状图，前3个区域使用亮蓝色，其余使用深蓝色

#### 5. AlarmList - 实时告警信息

- **图表类型**: **非ECharts图表**，纯Vue组件实现
- **数据类型**: **静态Mock数据** + 定时生成新数据

```typescript
const MESSAGES = {
  critical: ['线路过载...', '变压器温度异常...', ...],
  warning: ['功率因数低于0.9...', '电压偏差超过...', ...],
  info: ['设备定期巡检提醒...', '电容器组投入运行...', ...]
}

function randomAlarm(): Alarm {
  const levels: Alarm['level'][] = ['critical', 'warning', 'warning', 'info', 'info', 'info']
  // ...
}
```

- **更新频率**: 每4秒生成一条新告警
- **特性**: 自动滚动列表，使用Element Plus的Badge和Tag组件

#### 6. LineStatusChart - 输电线路负荷率

- **图表类型**: `bar`（水平柱状图）
- **数据类型**: **静态Mock数据** + 随机波动

```typescript
const LINES = [
  '昌顺220kV', '顺通110kV', '通兴220kV',
  '兴房110kV', '房密110kV', '密延110kV', '延石500kV'
]

function getLoadRates() {
  return LINES.map(() => Math.round(45 + Math.random() * 50))
}
```

- **更新频率**: 每5秒更新一次
- **特性**: 水平条形图，根据负荷率显示不同颜色（>90%红色，>75%橙色，其他蓝色）

### 2.3 图表组件数据汇总

| 组件 | 图表类型 | 数据来源 | 更新频率 |
|------|---------|---------|---------|
| PowerLoadChart | line + scatter | Mock + 随机波动 | 5秒 |
| GenerationPie | pie | Mock + 随机波动 | 8秒 |
| SubstationStatus | 纯Vue组件 | Mock + 随机波动 | 6秒 |
| EnergyConsumption | bar | Mock + 随机波动 | 6秒 |
| AlarmList | 纯Vue组件 | Mock + 定时生成 | 4秒 |
| LineStatusChart | bar（水平） | Mock + 随机波动 | 5秒 |

**结论**: 所有6个组件都使用**静态Mock数据**，通过 `Math.random()` 添加随机波动来模拟实时数据效果，**没有真实的API数据请求**。

---

## 三、大屏响应式适配方案

### 3.1 整体布局架构

大屏采用 **CSS Grid + Flexbox** 布局：

```css
.dashboard-main {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;  /* 左:中:右 = 1:2:1 */
  gap: 8px;
  padding: 8px;
  overflow: hidden;
  min-height: 0;
}
```

布局结构：
- **左侧面板**: 1份宽度（约25%）
- **中间地图**: 2份宽度（约50%）
- **右侧面板**: 1份宽度（约25%）

### 3.2 响应式断点设计

项目定义了3个主要断点：

| 断点 | 目标设备 |
|------|---------|
| `max-width: 480px` | 手机小屏 |
| `max-width: 900px` | 平板/小屏电脑 |
| `min-width: 1600px` | 大屏显示器 |
| `min-width: 2560px` | 超大屏/4K显示器 |

### 3.3 各断点适配策略

#### 3.3.1 小屏适配（max-width: 900px）

```css
@media (max-width: 900px) {
  .dashboard {
    height: auto;
    min-height: 100vh;
    overflow-y: auto;
  }

  /* Header改为垂直布局 */
  .dashboard-header {
    height: auto;
    flex-direction: column;
    align-items: center;
  }

  /* 主内容改为单列布局 */
  .dashboard-main {
    grid-template-columns: 1fr;
    overflow: visible;
  }
}
```

**适配要点**:
- 主布局从三列变为单列
- 头部从水平排列改为垂直堆叠
- 地图高度固定为420px

#### 3.3.2 手机适配（max-width: 480px）

```css
@media (max-width: 480px) {
  .main-title {
    font-size: 16px;
    letter-spacing: 1px;
  }

  .title-en {
    display: none;  /* 隐藏英文标题 */
  }

  .weather-row {
    display: none;  /* 隐藏天气信息 */
  }
}
```

**适配要点**:
- 进一步缩小字体
- 隐藏次要信息（英文标题、天气）
- 地图高度固定为320px

#### 3.3.3 大屏适配（min-width: 1600px）

```css
@media (min-width: 1600px) {
  .dashboard-header {
    height: 108px;
    padding: 0 40px;
  }

  .main-title {
    font-size: 34px;
  }

  .dashboard-main {
    gap: 12px;
    padding: 12px;
  }
}
```

**适配要点**:
- 增大头部高度和字体
- 增加间距
- 整体放大1.1倍左右

#### 3.3.4 超大屏适配（min-width: 2560px）

```css
@media (min-width: 2560px) {
  .dashboard-header {
    height: 130px;
    padding: 0 60px;
  }

  .main-title {
    font-size: 42px;
  }

  .dashboard-main {
    gap: 16px;
    padding: 16px;
  }
}
```

**适配要点**:
- 进一步增大所有元素尺寸
- 整体放大1.3倍左右

### 3.4 保证布局不变形的核心策略

#### 1. 使用 `min-height: 0` 防止Flexbox溢出

```css
.panel-section {
  display: flex;
  flex-direction: column;
  min-height: 0;  /* 关键：允许flex item收缩 */
  overflow: hidden;
}
```

#### 2. ECharts图表自动resize

所有图表组件都监听窗口resize事件：

```typescript
function handleResize() { chart?.resize() }

onMounted(() => {
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
```

#### 3. 使用相对单位

- 字体大小使用 `rem` 和 `px` 混合
- 间距使用 `px` 但在媒体查询中调整
- 图表容器使用百分比或flex布局

#### 4. 图表容器自适应

```css
.chart-body {
  flex: 1;
  min-height: 0;  /* 允许收缩 */
  width: 100%;
}
```

#### 5. 固定比例布局

中间地图区域使用相对比例（grid 2fr），保证在不同宽度下保持视觉平衡。

### 3.5 响应式适配总结

| 分辨率范围 | 布局方式 | 特殊处理 |
|-----------|---------|---------|
| < 480px | 单列 | 隐藏次要信息，最小字体 |
| 480px - 900px | 单列 | 头部堆叠，地图固定高度 |
| 900px - 1600px | 三列（默认） | 标准布局 |
| 1600px - 2560px | 三列 | 增大间距和字体 |
| > 2560px | 三列 | 最大尺寸，适合4K屏 |

---

## 四、技术栈总结

| 类别 | 技术 |
|------|------|
| 框架 | Vue 3 + Composition API |
| 语言 | TypeScript |
| 图表 | ECharts 5 + ECharts-GL |
| UI组件 | Element Plus |
| 样式 | CSS3 + CSS Variables |
| 构建 | Vite |

