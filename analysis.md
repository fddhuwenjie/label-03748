# Vue3电力驾驶舱大屏项目源码分析

## 1. 3D北京地图实现分析

### 实现技术栈
**MapView** 组件中的3D北京地图基于 **ECharts + ECharts-GL** 技术栈实现：
- 主库：`echarts` (v5.x) - 提供基础图表能力
- 3D扩展：`echarts-gl` (v2.x) - 提供WebGL渲染的3D地理可视化
- GeoJSON：使用北京地理边界数据文件进行地图轮廓渲染

### 核心配置说明

#### 相机参数配置 (viewControl)
```typescript
viewControl: {
  distance: 110,      // 相机距离
  alpha: 42,          // 上下仰角
  beta: 2,            // 左右旋转角
  minAlpha: 15,       // 最小仰角
  maxAlpha: 75        // 最大仰角
}
```
使用透视投影相机，通过 `alpha/beta/distance` 控制视角，支持用户交互旋转缩放。

#### 光照系统配置 (light)
```typescript
light: {
  main: {
    intensity: 1.4,           // 主光强度
    shadow: true,             // 启用阴影
    shadowQuality: 'high',    // 阴影质量
    alpha: 55,                // 主光仰角
    beta: 10                  // 主光方位角
  },
  ambient: { intensity: 0.5 },  // 环境光强度
  ambientCubemap: {
    exposure: 1,
    diffuseIntensity: 0.5
  }
}
```

#### 材质属性 (realisticMaterial)
```typescript
realisticMaterial: {
  roughness: 0.4,   // 粗糙度 0=镜面 1=漫反射
  metalness: 0.5    // 金属度 0=塑料 1=金属
}
```
采用 PBR (Physically-Based Rendering) 真实感渲染材质。

---

## 2. ECharts图表组件分析

| 图表组件 | ECharts图表类型 | 数据源 | 核心实现说明 |
|---------|----------------|--------|-------------|
| **PowerLoadChart** | `line` + `scatter` 混合系列 | 静态Mock数组 `BASE_LOADS` | x轴：24小时时间轴<br>y轴：200min起万kW值<br>2组series：1条smooth折线+面积填充 + 1个scatter当前时刻圆点<br>smooth:0.4, symbol:none, shadowBlur光影效果 |
| **GenerationPie** | `pie` 环形饼图 | 静态Mock + 定时更新 | 6种SOURCES能源：火电/风电/光伏/水电/核电/其他能源<br>radius: ['38%', '68%']<br>itemStyle.borderWidth:2px 边框间隔<br>label.show:false 无标签（左侧legend显示）<br>borderColor: '#050a1a' 深色背景间隔 |
| **SubstationStatus** | ❌ 纯Vue组件 (无ECharts) | 静态变电站数组 | 8个STATION_NAMES变电站列表：<br>• 每行1个条目：名称(带status-dot指示灯) + 带背景条的进度条(load%) + 电压等级(kv)<br>• 底部汇总：正常/预警/故障 三色圆点计数 |
| **EnergyConsumption** | `bar` 单组柱状图 | 静态Mock数组 `DISTRICTS` | x轴：前8个北京区县名称<br>y轴：负荷值<br>单组series，barWidth:'60%'<br>前3个颜色 #00d4ff 渐变，后5个 #0080ff 渐变 |
| **AlarmList** | ❌ 纯Vue + Element Plus | 静态告警对象数组 | el-table + 复合样式：<br>• el-tag type：颜色区分告警级别<br>• status-dot：脉冲动画运行/预警/离线状态<br>• badge：红色徽标计数<br>• 时间轴格式化显示 |
| **LineStatusChart** | `bar` 横条柱状图 | 静态线路数组 | LINES=7条具体线路：['昌顺220kV', '顺通110kV', '通兴220kV', '兴房110kV', '房密110kV', '密延110kV', '延石500kV']<br>yAxis类目显示具体线路名<br>barWidth:10px细条<br>label:右侧显示 {c}% 负荷率数值<br>loadRate颜色：90%+红 75%+橙 其余蓝<br>水平方向LinearGradient彩色值条 |

> 📌 **数据特征总结**：
> - 所有图表均使用**本地静态数组**作为基础数据源
> - 通过 `setInterval/math.random()` 实现数值周期性波动模拟动态效果
> - 无任何后端 `axios/fetch/websocket` 数据请求
> - 更新周期：`1000ms ~ 8000ms` 按需配置

---

## 3. 大屏响应式适配方案

### **核心适配策略：Mobile First + 渐进增强**

#### 1) 全局基础适配
**文件**：`src/style.css`
```css
html, body, #app {
  width: 100%;
  height: 100%;
  overflow: hidden;     /* PC大屏：锁定滚动 */
}

/* <= 900px 小屏切换 */
@media (max-width: 900px) {
  html, body, #app {
    height: auto;
    overflow: auto;      /* 移动端：允许垂直滚动 */
  }
}
```

#### 2) App.vue 三栏 → 单栏 响应式断点

| 分辨率区间 | 布局行为 | grid-template-columns | 核心适配 |
|-----------|---------|----------------------|---------|
| **> 900px** PC桌面 | 三列式等比分栏 | `1fr 2fr 1fr` | LeftPanel + 中心3DMap + RightPanel<br>flex:1 垂直等分 |
| **481px ~ 900px** 平板 | Header堆叠居中 → 单列滚动 → Footer堆叠 | `1fr` | order:-1 将header-center前置<br>overflow:visible 解除高度锁定 |
| **<= 480px** 手机 | 极致精简版 | `1fr` | 隐藏英文字幕 `.title-en`<br>隐藏天气 `.weather-row`<br>减小gap压缩留白 |
| **>= 1600px** 2K屏 | 等比放大 | 维持原比例 | height: 108px Header<br>font-size: 放大 ~10% |
| **>= 2560px** 4K/8K屏 | 细节增强 | 维持原比例 | height: 130px Header<br>所有字体 ~+35% 物理像素 |

#### 3) Panel 组件高度自适应关键代码
```css
.dashboard-main {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 8px;
  padding: 8px;
  overflow: hidden;
  min-height: 0;    /* 关键点：允许flex子项压缩 */
}

.panel-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
  overflow: hidden;
  min-height: 0;    /* 关键点：flex布局防变形核心 */
}
```

#### 4) 分辨率适配保障机制

| 技术手段 | 防止变形原理 | 应用场景 |
|---------|------------|---------|
| **min-height: 0** | 覆盖flex默认 `min-height:auto`，允许容器按需压缩 | grid + flex混排场景 |
| **overflow:hidden ↔ auto** | PC端锁定窗口保证无滚动；移动端解放高度适配流 | 全局html/body/.dashboard |
| **fr 相对单位** | viewport宽度变化时自动分配比例宽度 | 三栏布局 `1fr 2fr 1fr` |
| **尺寸断点同步** | LeftPanel/RightPanel/App.vue 所有文件 900px/1600px/2560px 断点统一 | 跨组件保持适配行为一致 |
| **固定高度兜底** | @media(<=900px) `.flex-grow` flex:none + height:220px | 移动端echarts容器初始化渲染 |