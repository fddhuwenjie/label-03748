# 中国电网驾驶舱 - China Power Grid Cockpit

## How to Run

### Docker 启动（推荐）

> 要求：已安装 Docker 和 Docker Compose，支持 ARM 和 X86 平台

```bash
# 构建并启动（后台运行）
docker-compose up --build -d

# 查看运行状态
docker-compose ps

# 查看日志
docker-compose logs -f frontend

# 停止服务
docker-compose down
```

启动后访问：**http://localhost:8081**

### 本地开发启动

> 要求：Node.js >= 18

```bash
# 进入前端目录
cd Frontend

# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview
```

开发服务器默认地址：**http://localhost:5173**

---

## Services

| 服务名 | 描述 | 端口 | 技术栈 |
|--------|------|------|--------|
| frontend | 电网可视化大屏前端 | 8081 (Docker) / 5173 (Dev) | Vue 3 + TypeScript + Vite + Element Plus |

**服务说明：**

- **frontend**：纯前端静态服务，使用 Nginx 提供服务（Docker 环境），Vite Dev Server（开发环境）。所有数据为前端模拟数据，无需后端服务。

---

## 测试账号

本项目为纯前端可视化大屏，**无需登录，无需账号**，直接访问即可使用。

访问地址：
- Docker 部署：http://localhost:8081
- 本地开发：http://localhost:5173

---

## 题目内容

帮我用vue3+ts+vite+element plus搭建一个中国电网可视化大屏系统，大屏采用系统名称为"中国电网驾驶舱"，左侧和右侧分别是电网相关的一些统计项目，中间是北京市3D地图，整个系统风格采用酷炫科技风格，左侧和右侧以及地图模块都采用组件式开发，方便替换和维护。

---

## 项目介绍

**中国电网驾驶舱**是一个基于 Vue 3 + TypeScript + Vite + Element Plus 构建的电力系统可视化大屏系统，采用酷炫科技风格设计，为电网调度人员提供全面的电网运行态势感知能力。

### 功能模块

#### 顶部状态栏
- 实时时间显示（时:分:秒）
- 电网系统频率监控（50.00Hz）
- 系统总负荷、在线站点数
- 天气信息展示

#### 左侧面板（LeftPanel）
- **KPI 卡片**：今日总发电量、总用电量及趋势
- **实时电网负荷曲线**：24小时负荷曲线，实时滚动更新，当前时刻高亮标注
- **发电能源结构**：火力/水力/核能/风力/光伏/其他新能源占比饼图，含图例
- **关键变电站状态**：8座关键变电站实时负荷率、运行状态（正常/预警/故障）

#### 中央地图（MapView）
- **北京市电网态势感知图**：基于 ECharts + GeoJSON 的北京行政区划底图
- 500kV/220kV/110kV 三级变电站标注（不同颜色和大小区分）
- 实时输电线路流向动画效果（动态箭头）
- 线路负荷颜色区分（正常/预警/过载）
- 多模式切换（电网/负荷/告警视图）
- 实时统计：变电站数、输电线路数、供电面积、供电用户数

#### 右侧面板（RightPanel）
- **KPI 卡片**：电网可靠率、综合线损率（带环形进度条）
- **实时告警信息**：自动滚动告警列表，分紧急/预警/通知三个级别，实时新增
- **输电线路负荷率**：横向条形图展示各主要线路负荷率，超限变色预警
- **各区用电量统计**：北京各行政区用电量柱状图

#### 底部信息栏
- 今日发电量、用电量、当前告警数量实时更新

### 技术架构

```
src/
├── components/
│   ├── LeftPanel/          # 左侧面板（组件化）
│   │   ├── index.vue       # 左侧面板容器
│   │   ├── PowerLoadChart.vue    # 实时负荷曲线
│   │   ├── GenerationPie.vue     # 发电能源结构
│   │   └── SubstationStatus.vue  # 变电站状态
│   ├── RightPanel/         # 右侧面板（组件化）
│   │   ├── index.vue       # 右侧面板容器
│   │   ├── AlarmList.vue         # 告警信息列表
│   │   ├── LineStatusChart.vue   # 线路负荷图
│   │   └── EnergyConsumption.vue # 用电量统计
│   └── MapView/            # 地图模块（组件化）
│       └── index.vue       # 北京电网地图
├── App.vue                 # 主布局
├── main.ts                 # 入口
└── style.css               # 全局样式
```

### 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| Vue 3 | ^3.4 | 前端框架 |
| TypeScript | ^5.4 | 类型系统 |
| Vite | ^5.2 | 构建工具 |
| Element Plus | ^2.6 | UI 组件库 |
| ECharts | ^5.4 | 数据可视化 |
| echarts-gl | ^2.0 | WebGL 3D 地图渲染 |
| Nginx | alpine | 生产环境 Web 服务器 |

### 设计风格

- **色彩**：深蓝科技背景（#050a1a），青色主色调（#00d4ff），霓虹绿辅色（#00ff88）
- **动效**：扫描线动画、脉冲发光、数据流动效果
- **布局**：左1/4 + 中1/2 + 右1/4 三栏自适应布局
- **字体**：等宽字体数字显示，营造科技感
