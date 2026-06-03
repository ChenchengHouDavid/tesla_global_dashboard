---
name: tesla-finance-dashboard-skill
description: Tesla Germany financial policy vs sales interactive HTML dashboard with regression analysis
metadata: 
  node_type: memory
  type: reference
  originSessionId: c8327319-8092-4f28-82d8-8457bf05d236
---

Skill located at: `~/Desktop/work/🦞/siklls/tesla_finance_dashboard.md`

Contains:
- Data processing script: `tesla_finance_data_process.py`
- HTML generation script: `tesla_finance_gen_html.py`
- Dashboard template: `tesla_finance_dashboard_template.html`

Key features:
- 6-tab interactive HTML dashboard (ECharts)
- Regression: 销量=β₀+β₁×月租+β₂×月供+β₃×净价+β₄×D_马斯克+β₅×D_过渡期+ε
- Financial attribution with real leasing/loan share data
- M3 and MY split, 高配/低配 grouping
- 0息 markPoint labels on interest rate charts
- Unified Y-axis ranges between M3 and MY charts
