# Vue3电力驾驶舱大屏项目源码分析报告

---

## 问题1：MapView组件中的3D北京地图实现方案

### 核心技术栈
| 项 | 说明 |
|-----|------|
| **渲染引擎** | ECharts + ECharts GL (WebGL渲染) |
| **地图数据** | GeoJSON格式 (`/beijing.json`) |
| **初始化方式** | `echarts.init(chartRef.value, undefined, { renderer: 'webgl' })` |

### 相机参数配置 (viewControl)
```typescript
viewControl: {
  distance: 110,           // 相机距离
  alpha: 42,               // 仰角
  beta: 2,                 // 方位角
  minAlpha: 15,            // 最小仰角限制
  maxAlpha: 75,            // 最大仰角限制
  panSensitivity: 0,       // 平移敏感度(禁用平移)
  rotateSensitivity: 0.8   // 旋转敏感度
}
```

### 光照配置 (light)
```typescript
light: {
  main: {
    intensity: 1.4,        // 主光源强度
    shadow: true,          // 启用阴影
    alpha: 55,             // 光源仰角
    beta: 10               // 光源方位角
  },
  ambient: {
    intensity: 0.5         // 环境光强度
  }
}
```

### 材质配置
```typescript
shading: 'realistic',              // 真实感渲染
realisticMaterial: {
  roughness: 0.4,          // 粗糙度
  metalness: 0.5           // 金属度
}
```

### Series类型汇总
共5种叠加图层：
1. `map3D` - 北京地图面
2. `bar3D` - 3D柱状图（各区域数值）
3. `lines3D` - 3D飞线效果
4. `scatter3D` - 点标记x3（不同symbol和大小）

---

## 问题2：左右面板6个图表组件分析

| 组件名称 | ECharts图表类型 | 数据来源 | 更新策略 |
|---------|---------------|----------|---------|
| **PowerLoadChart.vue** | `line + scatter` 组合折线散点图 | 静态mock | `setInterval` 每5秒模拟更新 |
| **GenerationPie.vue** | `pie` 环形图<br>(radius: ['38%', '68%']) | 静态mock | `setInterval` 每8秒模拟更新 |
| **SubstationStatus.vue** | ❌ 纯Vue+CSS，**不使用ECharts** | 随机生成 | `setInterval` 每4秒 |
| **EnergyConsumption.vue** | `bar` 垂直柱状图<br>(带LinearGradient渐变) | 静态mock+随机扰动 | `setInterval` 每6秒 |
| **AlarmList.vue** | ❌ Element Plus，**不使用ECharts**<br>(ElBadge + ElTag) | 静态ALARMS数组 | `setInterval` 每4秒新增，JS控制平滑滚动 |
| **LineStatusChart.vue** | `bar` 水平条形图<br>(颜色随loadRate动态变化) | 静态mock+随机扰动 | `setInterval` 每7秒 |

### 颜色动态变化规则 (LineStatusChart)
- loadRate > 90 → #ef4444 (红色)
- loadRate > 75 → #f97316 (橙色) 
- 默认 → #3b82f6 (蓝色)

### ✅ 重要结论
**所有组件全部使用静态mock数据**，使用 `setInterval` (4s-8s不等) 定时器模拟动态数据效果。
**代码审计确认**：6个组件源码中**无任何 fetch / axios 网络请求**，完全为离线演示版本。

---

## 问题3：整个大屏的响应式适配方案

### 核心布局
App.vue 采用 CSS Grid 三栏等高布局：
```css
.main-content {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;  /* 左:中:右 = 1:2:1 */
  gap: 1rem;
  min-height: 0;
}
```

### 断点策略（4级Media Query）
| 分辨率 | 适配规则 |
|--------|---------|
| **≤ 480px** | 极致小屏：<br>- 隐藏英文标题 `.title-en`<br>- 隐藏天气行 `.weather-row` |
| **≤ 900px** | 通用小屏：<br>- Header flex-direction: column（堆叠）<br>- `.main-content` 改为 `grid-template-columns: 1fr`（单栏竖向滚动）<br>- `.page-container` 改为 `height: auto` + `overflow: auto`<br>- Footer flex-direction: column |
| **≥ 1600px** | 标准大屏：<br>- 字体放大1.1倍 (`font-size: 18px`)<br>- `.title-main` 高度增至 70px<br>- gap 增至 `1.5rem` |
| **≥ 2560px** | 2K/4K超高清：<br>- 字体放大至 20px<br>- `.title-main` 高度增至 80px<br>- gap 进一步放大 |

### 核心技术点
1. **CSS变量**：全系统使用 design token (`var(--text-muted)`, `var(--color-primary)`, `var(--glow-color)`)
2. **Viewport单位**：标题使用 `min(2.8vw, 3rem)` 实现流体响应
3. **overflow切换**：大屏固定视口(overflow:hidden)，小屏自动滚动(overflow:auto)
4. **弹性布局降级**：900px以下三栏变单栏，避免内容挤压
5. **无障碍增强**：所有组件完整ARIA属性（role, aria-label, aria-live, aria-busy）

---

## 文件结构索引
```
/src
  /components
    /MapView/index.vue        # 3D北京地图（echarts+echarts-gl）
    /LeftPanel
      /PowerLoadChart.vue     # line+scatter折线图
      /GenerationPie.vue      # pie环形图
      /SubstationStatus.vue   # 纯CSS进度条
    /RightPanel
      /EnergyConsumption.vue  # 垂直bar图
      /AlarmList.vue          # Element Plus滚动列表
      /LineStatusChart.vue    # 水平bar图
  /App.vue                    # Grid布局+响应式Media Query
  /style.css                  # CSS全局变量定义
```

**分析完成时间**：2025年
