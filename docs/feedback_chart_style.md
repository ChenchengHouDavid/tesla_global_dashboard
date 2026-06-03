---
name: Tesla chart style and terminology preferences
description: User preferences for Tesla Germany chart terminology, colors, and alignment standards
type: feedback
originSessionId: 94b5a5df-75f2-4519-9ea1-0c9150dbff6d
---

## 用语规范
- 不要用 "Juniper"，用 "新款MY" 或 "新款MY发布"
- 不要用 "930调价"，用 "MY标准版降价€5,000"

**Why:** 用户认为这些术语对读者不友好，"Juniper"是内部代号，"930调价"含义模糊。

**How to apply:** 在图表标注和飞书文档中，统一使用面向消费者的描述语言。

## 颜色规范
- M3 相关标注用红色（#E82127）
- MY 相关标注用蓝色（#3b82f6）
- MY降价/补贴用绿色（#10b981）
- 月租变化用橙色（#f59e0b）
- 政府政策用玫红色（#e11d48）

**Why:** 用户要求颜色与车型/政策类型一致，便于快速识别。

**How to apply:** 在 tesla_germany_08_enhanced.py 的标注数组中，每个 annotation 的颜色参数必须遵循此规范。

## 对齐标准
- 季度总结块必须与柱状图精确对齐（"无差别对齐"）
- 不能有视觉偏差

**Why:** 用户对精确度要求高，哪怕是几像素的偏差也能察觉。

**How to apply:** 使用 PIL/numpy 直接扫描渲染后的图片测量柱状图位置，不要依赖 matplotlib 坐标变换（bbox_inches='tight' 会引入偏移）。
