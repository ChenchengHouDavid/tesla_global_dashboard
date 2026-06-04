# 特斯拉金融地图看板 — Handoff 文档

**最后更新**: 2026-06-04
**当前阶段**: 开发完成。圆点缩放「持久错位」bug 已修复（georoam 防抖重绘）；「缩放全程丝滑跟随」为已知局限，暂不实现（见第五节）
**当前文件**: `tesla_dashboard.html` (~910 KB)
**GitHub Pages**: https://chenchenghouDavid.github.io/tesla_global_dashboard/

---

## 一、项目概述

将两个独立看板合并为统一 dashboard（`tesla_dashboard.html`）：

| 来源看板 | 技术 | 内容 |
|----------|------|------|
| 特斯拉金融合作伙伴与产品地图看板.html | ECharts 世界地图 | 24国金融机构/产品类型 |
| 特斯拉销量看板.html（外部文件） | Chart.js | 8区域 × 6年 Tesla BEV 销量 |

合并后有两个视图：
- **全球金融** 视图：左侧边栏 + ECharts 世界地图 + 多点集群圆点
- **销量总结** 视图：左侧边栏 + 纯 HTML/SVG 图表（柱状图/折线图/雷达图/热力图）

通过右上角 `view-toggle` 按钮切换，侧边栏始终保留。

---

## 二、当前文件结构

```
C:\Users\侯宸城\Downloads\特斯拉金融地图看板_交接包\
├── tesla_dashboard.html                          ← ★ 主文件 (910KB, 当前开发版)
├── 特斯拉金融合作伙伴与产品地图看板.html           ← 原始金融看板 (307KB, 早期版本)
├── 特斯拉金融合作伙伴与产品地图看板_v2.html        ← v2 迭代版 (880KB, 旧版)
├── 特斯拉金融合作伙伴与产品地图看板_0603.html      ← 0603 快照版 (862KB)
├── HANDOFF.md                                    ← 本文档
├── README.md                                     ← 部署/用户文档
├── data\
│   ├── 特斯拉主要金融合作伙伴与金融产品.xlsx       ← 金融原始数据
│   └── 世界地图.pptx                              ← 参考地图 (未直接使用)
└── docs\
    ├── feedback_chart_style.md                    ← 图表样式/用语规范
    ├── project_auto_finance_dashboard.md          ← 34国汽车金融看板项目记录
    ├── project_tesla_germany_chart_monthly.md     ← 德国月度图表任务
    └── tesla_finance_dashboard_skill.md           ← 金融政策看板 Skill

外部参考文件（不在本目录）:
C:\Users\侯宸城\Downloads\
├── 全球10年-25年数据.xlsx                          ← ★ 销量权威数据源 (Marklines, Tesla筛选)
├── 特斯拉金融地图看板_交接包\ 之外的 html 历史版本
└── tesla_dashboard.html → 已部署为 GitHub Pages (index.html)
```

**GitHub Pages 部署目录**: `C:\Users\侯宸城\Downloads\tesla_dashboard_deploy\`
```
tesla_dashboard_deploy\
├── index.html                    ← 即 tesla_dashboard.html 的副本 (重命名)
├── README.md
├── HANDOFF.md
├── 特斯拉金融合作伙伴与产品地图看板.html
├── 特斯拉金融合作伙伴与产品地图看板_v2.html
├── 特斯拉金融合作伙伴与产品地图看板_0603.html
├── data\
└── docs\
```

---

## 三、架构设计

### 整体布局

```
┌──────────────────────────────────────────────────────────────┐
│  主 header: view-toggle [全球金融] [销量总结]                  │
├──────────────┬───────────────────────────────────────────────┤
│  左侧边栏    │  主区域                                        │
│  (320px)    │  - 地图视图: ECharts 世界地图 + 多点集群圆点     │
│  始终可见    │  - 销量视图: 总结卡片 + 纯HTML/SVG图表          │
├──────────────┴───────────────────────────────────────────────┤
│  动态图例 / footer                                           │
└──────────────────────────────────────────────────────────────┘
```

### 数据流

```
金融数据: DATA[] (24国, 硬编码在 HTML 中)
  ├── n: 中文国名, r: EU/AP, p: 合作伙伴数组, pr: 产品数组, c: [经度,纬度]
  ├── C2G{}: 中文→English GeoJSON 名映射
  └── G2C{}: English→中文 反向映射

银行颜色模型 (从 DATA 动态计算):
  ├── _bankColorMap{}: 出现≥2次的银行→专属颜色
  ├── PARTNER_LEGEND[]: 图例数据 (Santander/Crédit Agricole/DNB/NF Fleet/未披露/其他银行)
  ├── COL_UNDISCLOSED: '#10b981' (未披露)
  └── COL_OTHER: '#94a3b8' (其他银行/灰色)

产品颜色模型 (固定):
  ├── PRODUCT_COLORS{}: 标准贷款/经营租赁/气球贷/融资租赁/无抵押贷款
  └── PRODUCT_LEGEND[]: 图例数据

银行介绍: BANK_INFO{} (24家机构, 无介绍的如Openbank已不存在)

销量数据 (两套):
  ├── SD{}: 46国明细 [2020...2025], 权威来源=全球10年-25年数据.xlsx 透视表
  ├── _OD{}: 8区域汇总 (亚太地区/北美/中西欧/北欧/东欧/南欧/亚非/南美)
  ├── S_C2G{}: 中文→English GeoJSON 名
  └── REGIONS{}: 区域→国家列表
```

### 圆点渲染架构（全球金融视图）

使用 ECharts **custom series** 渲染，非普通 scatter：

```javascript
// renderDots() 核心逻辑:
// 1. 根据当前 view (0=合作伙伴 / 1=产品) 确定每个国家的点集
// 2. 每国: N个小圆 (固定 DOT_R=9px) 按圆形排布
// 3. 每国: 一个不可见的外接圆作为点击热区 (共享 hit area)
// 4. 其他银行 (COL_OTHER 灰色) 注册到 _dotReg{} 用于 hover tooltip
// 5. 选中国家时, 其他银行的名称作为标签显示在圆点下方
```

### 销量总结图表渲染

完全使用纯 HTML/CSS/SVG（非 ECharts），因为 ECharts 在嵌套 flex/grid 容器中有容器尺寸为 0 的时序问题：

```javascript
renderBar(id, values, colors, labels, unit)      // CSS div 柱状图
renderLine(id, seriesData, xLabels)              // SVG 折线图 (测量容器宽度)
renderRadar(id, seriesData, labels)              // SVG 雷达图 (居中+HTML图例)
legendHTML(items)                                // 共用: wrapped HTML 图例
```

所有函数在 `renderSummaryMain()` 内定义，innerHTML 赋值后同步渲染。

---

## 四、已完成的所有功能

### 基础功能（已验证）
1. ✅ 地图替换 → Natural Earth 110m (177国, 819KB 内嵌 GeoJSON)
2. ✅ 配色重构 → 深蓝灰靛蓝专业配色 (`--bg:#0b1120`, `--accent:#6366f1`)
3. ✅ 侧边栏 → 统一左侧 320px，金融/产品 tab，概览/详情/银行三级视图
4. ✅ 多点集群圆点 → ECharts custom series，每国按银行/产品数显示多个小圆
5. ✅ 视图切换 → 合作伙伴视图 / 产品类型视图，圆点颜色+图例自动更新
6. ✅ 动态图例 → 根据视图显示不同图例（6项金融机构 / 5项金融产品）
7. ✅ 点击交互 → 点击国家圆点→侧边栏详情（金融信息+销量趋势迷你图）
8. ✅ 未披露机构识别 → 灰色圆点，hover 显示银行名，选中后内联显示
9. ✅ 银行介绍三级页 → 24家机构有详情页（机构定位+核心能力+合作市场）
10. ✅ 地图三级高亮 → 金融+销量=深蓝, 仅金融=深蓝, 仅销量=浅蓝, 无数据=暗色
11. ✅ 销量总结全部图表 → 柱状图/增长率图/热力图/对比表/折线图/雷达图/区域分析

### 数据修正（已验证）
12. ✅ 5国销量数据修正（SD vs 权威透视表对齐）：美国/加拿大/韩国/俄罗斯/巴西
13. ✅ 德国金融机构修正：Openbank → Crédit Agricole
14. ✅ 澳大利亚/韩国/马来西亚机构数据更新（Tesla自营 → 真实银行）
15. ✅ 所有 Tesla自营 → 未披露（冰岛/泰国）

### 部署
16. ✅ GitHub Pages 部署（https://chenchenghouDavid.github.io/tesla_global_dashboard/）
17. ✅ README.md 更新（反映当前项目状态）

---

## 五、圆点缩放 — 已修复的 bug + 一个已知局限

这一节区分两件**不同**的事：
- **(A) 持久错位 bug** — 已修复 ✅
- **(B) 缩放全程丝滑跟随** — 已知局限，暂不实现 ⚠️

---

### (A) 持久错位 bug —— 已修复 ✅

#### 原现象
在全球金融视图中，**选中一个国家后再滚轮缩放地图**，圆点可能出现大范围错位且**停下后不会自动恢复**。例如：
- 选中葡萄牙 → 缩放 → 点击西班牙 → 再缩放 → 部分圆点跑到错误位置（如香港的点出现在非洲）且留在那里

#### 根因
圆点用 ECharts **custom series** 渲染，`renderItem` 用 `api.coord()` 把经纬度转像素，依赖 geo 组件内部的像素变换矩阵。原代码**完全没有监听 `georoam` 事件**，所以缩放/拖拽后，圆点的最终状态从不保证用「稳定后的矩阵」重画过——一旦某次重画（如 `showDetail` 里 setOption 改 region 后立即调的 renderDots）撞上变换中间态，错误坐标就留下来不再纠正。

#### 修复（当前代码，`initMap()` 内）
监听 `georoam`，缩放/拖拽停止后防抖 90ms 强制重画一次圆点：
```javascript
chart.on('georoam',function(){
  clearTimeout(chart._dotTimer);
  chart._dotTimer=setTimeout(function(){
    if(currentView==='map')renderDots();
  },90);
});
```
**效果**：无论缩放中途渲染对错，手一停（90ms 后）必然用最终稳定矩阵重画，圆点精确归位。一行新增、零副作用，未触碰 `renderDots`/`renderItem`/`showDetail` 既有逻辑（`showDetail` 末尾原有的 `requestAnimationFrame(renderDots)` 保留）。

**已用 Playwright 真实 wheel 缩放验证**：缩放停止后所有圆点（欧洲各国 + 韩国/中国香港/泰国/菲律宾/马来西亚）精确贴回对应国家。

---

### (B) 缩放「全程丝滑跟随」—— 已知局限，暂不实现 ⚠️

#### 现象
上面的修复是「**停下即归位**」，不是「**全程跟随**」。**缩放进行中的那几帧**，地图已经在实时变换，但圆点还停在旧坐标（实测：放大过程中亚太圆点群会短暂飘到地图中部），手一停才整体跳回正确位置。即缩放过程中圆点会「滞后/跳动」一下，不是丝滑跟着地图缩放。

#### 根因（为什么做不到全程跟随）
ECharts 里 **custom series 圆点**和 **geo 地图**是两个独立图层。缩放时 geo 地图由 ECharts 内部逐帧实时变换，但 custom series 的 `renderItem` **不会在 roam 的每一帧自动重跑**，所以圆点不逐帧跟动，只能靠事件结束后整体重画。要「全程跟随」必须让圆点每帧都重算坐标。

#### 实现「全程丝滑跟随」的方案（如未来要做）
- **方案 1（推荐路线，但改动大）**：放弃 custom series，改用 ECharts **原生 `scatter`/`effectScatter`** 系列（`coordinateSystem:'geo'`）。原生散点是引擎内建图层，缩放时逐帧自动同步变换，真正丝滑。
  - **代价**：现有的「每国一圈多个小圆按环形排布」「选中国家时在下方展开『其他银行』名称块」「灰色圆点 hover 显示银行名」这套自定义渲染（`renderDots`/`renderItem`/`_dotReg`）需全部重做，风险较高，可能影响已有交互。
- **方案 2（不推荐）**：`georoam` 不防抖 + 每帧 `requestAnimationFrame` 持续重跑 `renderDots`。理论可行，但每帧整体重建 custom series 会卡顿，且仍可能撞上变换中间态，性能与效果均不理想。

#### 当前决策
保留「停下即归位」方案（方案 A）。它一行代码、零副作用、彻底消除了「持久错位」这个真正的 bug。「全程丝滑」属于体验优化，需走方案 1 的较大重构，会动到选中展开 / hover 提示等已有功能——**当前不做**，待后续有需要时再评估。

#### 复现步骤（观察当前行为）
1. 打开 `tesla_dashboard.html`，确保在「全球金融」视图
2. 滚轮快速缩放地图 → **缩放过程中**圆点会短暂滞后/跳动（局限 B）
3. 停止缩放 ~90ms 后 → 圆点精确归位（bug A 已修复，不再持久错位）

---

## 六、函数速查表

### 地图相关
| 函数 | 作用 |
|------|------|
| `initMap()` | 注册地图 GeoJSON，初始化 ECharts geo + custom series，绑定 click/georoam/resize |
| `renderDots()` | 重建多点集群散点（按当前 view 决定颜色），注册其他银行 hover 数据 |
| `hlMap(cn, on)` | hover 高亮地图区域（仅在未选中状态下生效） |
| `bankColor(n)` | 返回银行颜色（专属色 或 COL_OTHER 灰色） |
| `productColor(n)` | 返回产品颜色 |
| `hasBankInfo(n)` | 检查银行是否有介绍页 |

### 侧边栏
| 函数 | 作用 |
|------|------|
| `showOverview()` | 金融概览（合作伙伴/产品分布 + 国家列表） |
| `showDetail(d)` | 选中国家详情（金融信息 + 销量趋势迷你图 + renderDots 刷新选中状态） |
| `showBank(name)` | 银行介绍三级页（定位 + 能力 + 合作市场） |
| `openCountry(name)` | 从银行页直接跳转到某国详情（绕过 toggleCountry 防抖） |
| `goBack()` | 三级导航的返回按钮（bank→country→overview） |
| `toggleCountry(name)` | 点击国家 → 进入/退出详情 (650ms 防抖) |
| `switchView(v)` | 切换「合作伙伴」「产品类型」视图，重建圆点+更新图例 |
| `updateLegend()` | 更新底部图例（根据 view 显示不同列表） |

### 销量总结
| 函数 | 作用 |
|------|------|
| `renderSummarySidebar()` | 销量视图侧边栏（数据覆盖 + 按区域国家列表） |
| `renderSummaryMain()` | 渲染销量视图全部内容（卡片 + 图表 + 分析） |
| `renderBar(id, values, colors, labels, unit)` | CSS div 柱状图（测量容器高度自适应） |
| `renderLine(id, seriesData, xLabels)` | SVG 折线图（测量容器宽度自适应 + HTML 图例） |
| `renderRadar(id, seriesData, labels)` | SVG 雷达图（居中 + HTML 图例） |
| `legendHTML(items)` | 生成 wrapped HTML 图例 |
| `renderSalesTrend(c, elId)` | 侧边栏国家详情中 ECharts 迷你趋势图 |

### 视图切换
| 函数 | 作用 |
|------|------|
| `toggleView(v)` | 切换「全球金融」/「销量总结」，控制显示/隐藏 |

### 数据工具
| 变量/函数 | 作用 |
|-----------|------|
| `DATA[]` | 金融合作数据 (24 国, 硬编码) |
| `SD{}` | 46 国级销量 (与权威透视表对齐) |
| `_OD{}` | 8 区域汇总销量 (销量总结视图数据源) |
| `BANK_INFO{}` | 24 家银行介绍数据 |
| `_dotReg{}` | 其他银行圆点坐标注册表 (用于 hover tooltip) |
| `RT{}` / `R_ANALYSIS{}` / `R_COLORS{}` | ⚠️ 死代码 (早期残留, 未被调用) |
| `REGIONS{}` | 区域→国家列表 |
| `C2G{}` / `G2C{}` / `S_C2G{}` | 中文↔English GeoJSON 名映射 |
| `hasFin(c)` / `hasSales(c)` | 检查数据存在性 |
| `totalSales(c)` / `peakYear(c)` / `yoy(c)` | 销量统计工具 |

### 颜色常量
| 变量 | 用途 |
|------|------|
| `_bankColorMap{}` | 动态计算: ≥2次出现的银行→专属颜色 |
| `PARTNER_LEGEND[]` | 合作伙伴图例 (6项) |
| `PRODUCT_LEGEND[]` | 产品类型图例 (5项) |
| `COL_UNDISCLOSED` | '#10b981' 未披露 |
| `COL_OTHER` | '#94a3b8' 其他银行 (灰色) |
| `DOT_R` | 9px 固定小圆半径 |
| `regionColors{}` | 销量总结中 8 区域颜色 |

---

## 七、部署相关信息

### GitHub Pages
- **Repo**: https://github.com/ChenchengHouDavid/tesla_global_dashboard
- **Pages URL**: https://chenchenghouDavid.github.io/tesla_global_dashboard/
- **部署目录**: `C:\Users\侯宸城\Downloads\tesla_dashboard_deploy\`
- **主文件**: `index.html` (即 tesla_dashboard.html 的副本)
- **推送方式**: SSH (`git@github.com:ChenchengHouDavid/tesla_global_dashboard.git`)
- **注意**: 推送 README.md 不影响 Pages；推送 index.html 会影响 Pages

### 推送流程
```bash
# 更新部署目录
cp "C:\Users\侯宸城\Downloads\特斯拉金融地图看板_交接包\tesla_dashboard.html" \
   "C:\Users\侯宸城\Downloads\tesla_dashboard_deploy\index.html"

# 推送
cd "C:\Users\侯宸城\Downloads\tesla_dashboard_deploy"
git add index.html
git commit -m "描述"
git push
```

---

## 八、数据权威来源

### 销量数据
- **权威源**: `C:\Users\侯宸城\Downloads\全球10年-25年数据.xlsx` 的「透视表」工作表
- **筛选条件**: 整车厂/品牌 = Tesla, 级别 = (多项), 车种 = (多项)
- **数据范围**: 46国 × 6年 (2020-2025), 2025 总计 = 1,835,598
- **两套数据必须与此透视表一致**:
  - `SD{}` (46国明细) — 已验证 100% 一致
  - `_OD{}` (8区域汇总) — 已验证 100% 一致

### 金融数据
- **来源**: `data/特斯拉主要金融合作伙伴与金融产品.xlsx`
- **硬编码**: `DATA[]` 数组 (24国)
- **德国合作伙伴**: Crédit Agricole + Santander（已修正，原为 Openbank + Santander）

---

## 九、继续工作的指引

### 圆点缩放（已修复持久错位；全程跟随为已知局限）
1. 阅读第五节：持久错位 bug 已用 `georoam` 防抖重绘修复 ✅
2. 若未来需要「缩放全程丝滑跟随」，见第五节 (B) — 推荐方案 1（改用原生 scatter，改动较大）
3. 任何改动后同步推送到 GitHub Pages 部署目录（见第七节）

### 添加新国家/银行
1. 在 `DATA[]` 中添加国家条目
2. 如有银行介绍，在 `BANK_INFO{}` 中添加
3. 检查 `C2G{}` 中是否有对应的 GeoJSON 国名映射

### 更新销量数据
1. 更新 `全球10年-25年数据.xlsx` 透视表
2. 同步更新 `SD{}` 和 `_OD{}` 中的数据
3. 验证 2025 总计 = 1,835,598 (或新总计)

### 调试地图相关问题
- 地图点击: `chart.on('click', ...)` 在 `initMap()` 中
- 地图缩放: `chart.on('georoam', ...)` 已在 `initMap()` 绑定 — 防抖 90ms 后重画圆点（修复持久错位）
- 自定义系列: `renderDots()` 中的 `renderItem` 函数
- 地图区域高亮: `hlMap()` 和各处 `chart.setOption({geo:[{regions:regs}]})`
