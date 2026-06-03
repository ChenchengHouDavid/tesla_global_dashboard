---
name: Monthly Tesla Germany chart + financial data crawling task
description: Every month: crawl Tesla DE financial products, update chart, send to Feishu with analysis
type: project
originSessionId: 94b5a5df-75f2-4519-9ea1-0c9150dbff6d
---

每月初需完成以下任务，私信发给用户：

## 脚本文件

| 文件 | 作用 | 输出 |
|------|------|------|
| `tesla_germany_08_enhanced.py` | 主图（柱状图+市占率+利率标注+政策时间线） | `tesla_germany_09_final.png` |
| `tesla_germany_final_with_quarterly.py` | 在主图下方添加季度总结块（PIL合成） | `tesla_germany_final_with_quarterly.png` |
| `tesla_germany_quarterly.py` | 独立季度对比图（分组柱状图） | `tesla_germany_quarterly.png` |
| `tesla_germany_chart2_comparison.py` | 金融产品矩阵+竞品对比表 | `tesla_germany_chart2_comparison.png` |
| `tesla_germany_chart3_rates.py` | 利率变迁时间线+关键节点表 | `tesla_germany_chart3_rates.png` |

**飞书文档ID:** `ZYPJdEi7woZuAjxl1W2cErvnnCf`

## 步骤

### 1. 爬取特斯拉德国全量金融产品数据
从 tesla.com/de_de 获取：
- **各车型售价**（M3/MY 低配/高配）
- **月供矩阵**：按 km数 × 期限数的气球贷月供
- **月租矩阵**：按 km数 × 期限数的融资租赁月租
- **对客利率**（气球贷/融资租赁分别）

### 2. 从飞书获取最新金融政策变化
- 气球贷利率变化
- 月租变化
- 竞品对比数据
- 政府补贴政策变化

### 3. 从 S&P Global 拉取最新销量数据
参考 reference_sp_global_market_insight.md

### 4. 更新图表
更新 `tesla_germany_08_enhanced.py` 中的数据数组和注释，运行生成主图。
然后运行 `tesla_germany_final_with_quarterly.py` 合成季度总结。

### 5. 发送到飞书
飞书文档格式：
- **Callout高亮块**（4条总结）：MY金融政策变化、M3金融政策变化、季末冲量规律、金融手段效果
- **数据来源**脚注
- **图表**：手动粘贴（飞书API不支持程序化创建图片块）

## 重要注意事项

### 图表对齐
- matplotlib `bbox_inches='tight'` 会裁剪图片并偏移坐标，不能用 matplotlib transData 算像素位置
- 正确方法：直接扫描渲染后的图片找柱状图边缘（用 PIL/numpy 检测彩色像素）
- 季度总结块必须与柱状图精确对齐（无差别对齐）

### 飞书图片上传
- `doc_insert` / `doc_write` 的 markdown `![](url)` 语法**不能**上传本地图片
- `file://` 和 `localhost` URL 也不会自动上传
- 正确方法：用 Drive media API（curl + user access token）上传，得到 file_token
- 但无法通过 API 创建图片块（block_type=27），需要用户手动粘贴图片
- Token 存储位置：`~/.feishu-mcp-pro/auth.json`

### 用语规范
- 不要用 "Juniper"，用 "新款MY" 或 "新款MY发布"
- 不要用 "930调价"，用 "MY标准版降价€5,000"
- MY降价标注绿色，具体标注哪款+降幅
- M3用红色，MY用蓝色
