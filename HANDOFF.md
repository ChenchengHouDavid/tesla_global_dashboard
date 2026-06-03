# 特斯拉金融地图看板 — Handoff 文档

**最后更新**: 2026-06-03
**当前阶段**: 调试中 — 销量总结图表渲染问题未解决
**当前文件**: `tesla_dashboard.html` (848 KB)

---

## 一、项目概述

将两个独立看板合并为统一 dashboard：

| 原看板 | 类型 | 覆盖 |
|--------|------|------|
| `特斯拉金融合作伙伴与产品地图看板.html` | ECharts 世界地图 | 24国金融合作机构/产品 |
| `特斯拉销量看板.html` | Chart.js 图表 | 8区域 × 6年 Tesla 销量 |

合并后的 dashboard（`tesla_dashboard.html`）有两个视图：
- **全球金融** 视图：左侧边栏 + 主区域 ECharts 世界地图
- **销量总结** 视图：左侧边栏 + 主区域原销量分析内容

通过右上角 `view-toggle` 按钮切换，侧边栏始终保留。

---

## 二、当前文件结构

```
C:\Users\侯宸城\Downloads\特斯拉金融地图看板_交接包\
├── tesla_dashboard.html          ← ★ 主文件 (848KB, 当前开发版)
├── 特斯拉金融合作伙伴与产品地图看板.html   ← 原始金融看板 (307KB)
├── 特斯拉金融合作伙伴与产品地图看板_v2.html ← v2 迭代版 (834KB, 旧版)
├── README.md
├── .claude\plan.md              ← 早期设计计划
├── data\
│   ├── 特斯拉主要金融合作伙伴与金融产品.xlsx  ← 金融原始数据
│   ├── 世界地图.pptx            ← 参考地图 (未直接使用)
│   └── ne_110m.geojson          ← Natural Earth 地图 (可能已删除)
└── docs\
    ├── feedback_chart_style.md
    ├── project_auto_finance_dashboard.md
    ├── project_tesla_germany_chart_monthly.md
    └── tesla_finance_dashboard_skill.md
```

**外部参考文件**（不在本目录）:
```
C:\Users\侯宸城\Downloads\
├── 特斯拉销量看板.html            ← ★ 原销量看板 (Chart.js, 58KB, 完整能运行)
├── 全球10年-25年数据.xlsx        ← 销量原始数据 (Marklines)
└── 特斯拉销量看板.html            ← 同上
```

---

## 三、架构设计

### 整体布局

```
┌──────────────────────────────────────────────────────────────┐
│  主 header: view-toggle [全球金融] [销量总结]                  │
├──────────────┬───────────────────────────────────────────────┤
│  左侧边栏    │  主区域                                        │
│  (320px)    │  - 地图视图: ECharts 世界地图 + 散点标注         │
│  始终可见    │  - 销量视图: 总结卡片 + 图表 + 区域分析          │
│             │                                               │
├──────────────┴───────────────────────────────────────────────┤
│  图例 / footer                                               │
└──────────────────────────────────────────────────────────────┘
```

### 视图切换逻辑

```
toggleView('map')  → mapArea:block, summaryView:none, 侧边栏显示金融tab
toggleView('summary') → mapArea:none, summaryView:block, 侧边栏显示数据覆盖
```

### 数据流

```
金融数据: DATA[] (24国, 硬编码在 HTML 中)
  ├── n: 中文国名, r: EU/AP, p: 合作伙伴数组, pr: 产品数组
  ├── C2G: 中文→English GeoJSON 名映射
  └── G2C: English→中文 反向映射

销量数据: _OD{} (从原看板 data 对象 1:1 复制)
  ├── years: ['2020'...'2025']
  ├── regionOrder: ['亚太地区','北美','中西欧','北欧','东欧','南欧','亚非','南美']
  ├── regions[name]: {total[], growth[], color, countries{}}
  └── analysis[name]: {summary, factors[], trend}

国家级销量 (用于侧边栏迷你趋势图):
  SD{}: 按中文国名的 {country: [2020...2025]}, 从 Excel 提取
  S_C2G: 中文→English GeoJSON 名
```

---

## 四、已完成的优化

1. **地图替换**: 嵌入 GeoJSON → Natural Earth 110m (177国, 819KB)
2. **配色重构**: 黑+红 → 深蓝灰靛蓝专业配色 (`--bg:#0b1120`, `--accent:#6366f1`)
3. **侧边栏**: 统一左侧 320px，金融/产品 tab，概览/详情双视图
4. **国家圆点**: 按地区着色 (EU=琥珀, AP=青蓝)，统一 14px，点击触发详情
5. **数据清洗**: 银行名标准化 (Santander Consumer Bank→Santander)，产品名去英文
6. **标签防重叠**: 16个欧洲国家单独设置 label 偏移量 (`lblPos()`)
7. **交互改进**:
   - 点击国家→详情，再次点击 650ms 防抖→返回概览
   - `toggleCountry()` 统一入口，地图点击和侧边栏芯片共用
   - 侧边栏 hover → 地图高亮联动 (`hlMap()`)
8. **地图三级高亮**: 金融+销量=深蓝, 仅金融=深蓝, 仅销量=浅蓝 (#1a2d4a), 无数据=暗色
9. **销量总结侧边栏**: 数据覆盖统计 + 按区域分组国家列表 + 点击查看趋势
10. **销量总结主区域**: 4个汇总卡片 + 区域分析卡片 (8张)

---

## 五、当前未解决问题 — 销量总结图表不显示

### 现象

切换到「销量总结」视图后，汇总卡片和区域分析卡片正常显示，但以下区域**不显示**：
- 各区域销量柱状图 (sBar)
- 各区域增长率柱状图 (sGrowth)
- 热力图 (heatmapGrid)
- 各区域关键指标对比表格 (compTable)
- 各区域销量趋势折线图 (sTrend)
- 各区域增长率趋势折线图 (sGrowthTrend)
- 雷达图 (sRadar)

### 已尝试的修复
1. **ECharts + setTimeout(80ms)** — 失败（时序不确定）
2. **ECharts + requestAnimationFrame 双帧** — 失败（嵌套 flex/grid 中容器尺寸仍为 0）
3. **纯 HTML/CSS/SVG 渲染（当前）** — 等待测试

### 最新方案

**完全抛弃 ECharts (在 summary 视图中)**, 改用纯 HTML/CSS/SVG 渲染：

```javascript
// 三个 helper 函数在 renderSummaryMain 内部定义:
renderBar(id, values, colors, labels, unit)     // CSS div bar chart
renderLine(id, seriesData, xLabels, colors, W, H) // SVG line chart
renderRadar(id, seriesData, labels, W, H)         // SVG radar chart
```

所有图表数据在 `innerHTML` 赋值后**同步**完成渲染，不依赖任何异步回调。

Chart 容器结构（在 innerHTML 中）：
```
#summaryView
├── .sum-cards (汇总卡片，正常)
├── .sum-chart-row
│   ├── .sum-chart-box > .sum-chart#sBar      ← renderBar
│   └── .sum-chart-box > .sum-chart#sGrowth   ← renderBar
├── .heatmap-wrap > .heatmap-grid#heatmapGrid ← innerHTML
├── .sum-table-wrap > table#compTable         ← innerHTML
├── .sum-chart-row
│   ├── .sum-full-chart > .sum-chart#sTrend   ← renderLine
│   └── .sum-full-chart > .sum-chart#sGrowthTrend ← renderLine
├── .sum-full-chart > .sum-chart#sRadar       ← renderRadar
└── .sum-section > .rg-cards (区域分析，正常)
```

### 如果当前方案仍不显示

最可能的原因：
1. `.sum-chart` 高度 220px 在嵌套布局中没生效 → 检查 devtools computed height
2. `renderBar` / `renderLine` 中引用 `document.getElementById(id)` 时容器已存在但高度为 0 → 在 `innerHTML` 赋值后立即 `document.getElementById` 应该是同步可用的
3. 函数内部 JS 错误导致静默失败 → 打开浏览器 console 检查

可能需要的修复：
- 给 `.sum-chart` 加 `min-height: 220px`
- 在 `renderBar` 开头加 `console.log(id, document.getElementById(id))` 调试
- 检查 `var charts=[];` 之类的变量冲突（之前的 ECharts 代码残留）

---

## 六、函数速查表

### 地图相关
| 函数 | 作用 |
|------|------|
| `initMap()` | 注册地图 GeoJSON，初始化 ECharts geo + scatter |
| `renderDots()` | 重建散点数据（颜色按 view 和 region） |
| `hlMap(cn, on)` | hover 高亮地图区域 |
| `lblPos(name)` | 返回国家标签偏移量 |

### 侧边栏
| 函数 | 作用 |
|------|------|
| `showOverview()` | 金融看板概览（合作伙伴/产品分布 + 国家列表） |
| `showDetail(d)` | 选中国家详情（金融信息 + 销量趋势迷你图） |
| `toggleCountry(name)` | 点击国家 → 进入/退出详情 (650ms 防抖) |
| `switchView(v)` | 切换「合作伙伴」「产品类型」视图，更新圆点颜色 |
| `updateLegend()` | 更新底部图例 |

### 销量总结
| 函数 | 作用 |
|------|------|
| `renderSummarySidebar()` | 销量视图侧边栏（数据覆盖 + 按区域国家列表） |
| `renderSummaryMain()` | 渲染销量视图全部内容（卡片+图表+分析） |
| `renderSalesTrend(c, elId)` | 侧边栏国家详情中 ECharts 迷你趋势图 |
| `showSummaryOverview()` | 销量视图中返回概览，恢复侧边栏 |

### 视图切换
| 函数 | 作用 |
|------|------|
| `toggleView(v)` | 切换「全球金融」/「销量总结」，控制显示/隐藏 |

### 数据工具
| 函数/变量 | 作用 |
|-----------|------|
| `_OD` | 原看板 data 对象 (1:1 复制)，所有销量图表数据源 |
| `DATA[]` | 金融合作数据 (24 国) |
| `SD{}` | 国家级销量 (43 国) |
| `REGIONS{}` | 区域分组 |
| `C2G{}`, `G2C{}` | 中文↔English GeoJSON 名 |
| `S_C2G{}` | 销量国家中文→English GeoJSON 名 |
| `hasFin(cn)` | 检查是否有金融数据 |
| `hasSales(cn)` | 检查是否有销量数据 |
| `totalSales(cn)` | 该国总销量 |
| `peakYear(cn)` | 峰值年份 |
| `yoy(cn)` | 2025 同比 |

### 颜色常量
| 变量 | 用途 |
|------|------|
| `RC` | 地区颜色 (EU=#d97706, AP=#0891b2) |
| `PCC` | 合作伙伴颜色 (按类别) |
| `PPC` | 产品类型颜色 |
| `PCL`, `PPL` | 图例数据 |
| `regionColors{}` | 销量总结中 8 区域颜色 |

---

## 七、已知注意事项

1. **GeoJSON 内嵌**: `/*__GEOJSON__*/` 占位符，需 Python 注入（原文件 819KB）
2. **ECharts CDN**: 4级容灾加载（bootcdn→staticfile→bytecdn→jsdelivr），地图和迷你趋势图仍用 ECharts
3. **`_OD` 数据不直接用于地图映射**: 销量总结侧边栏使用 `_OD.regions` 的 countries 做国家列表，地图点击使用 `SD` 和 `S_C2G` 做映射
4. **区域分析中的 `Object.entries`**: 不被旧浏览器支持，但现代浏览器均可

---

## 八、如果要继续工作

1. **先测试当前状态**: 在浏览器打开 `tesla_dashboard.html`，切换到「销量总结」，检查图表是否显示
2. **如果不显示**: 打开 DevTools Console，检查是否有 JS 错误；检查 `.sum-chart` 容器是否有 computed height > 0
3. **如果图表不显示但无 JS 错误**: 可能是 `renderSummaryMain` 函数中的变量冲突或 `_OD` 引用问题。在 `renderBar`/`renderLine` 开头加 `console.log` 调试
4. **对齐数据**: 所有销量数据来自 `_OD` 对象，它 1:1 复制自原 `特斯拉销量看板.html` 的 `data` 变量。如果发现数据出入，对比这两个文件的 `total`/`growth`/`countries` 字段
