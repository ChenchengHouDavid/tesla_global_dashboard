---
name: auto-finance-dashboard
description: 34-country auto finance global dashboard project - HTML/Excel/Feishu outputs with deep research
metadata: 
  node_type: memory
  type: project
  originSessionId: 74a9d7a2-c65d-4fcd-9da7-70b3f6a2f3f4
---

## Global Auto Finance Dashboard Project

**Scope**: 34 countries across 7 clusters (Western Europe, Southern Europe, Nordic, Eastern Europe, Asia Pacific, Southeast Asia, Middle East)

**Why:** Internal strategic decisions + competitor analysis for automotive finance market entry

**How to apply:** When user references auto finance research, dashboard updates, or market analysis

### Output Files
- HTML Dashboard: `/Users/zhouzhou/output/auto_finance_global.html` + 34 country pages in `/output/countries/`
- Excel Report: `/Users/zhouzhou/output/auto_finance_global.xlsx`
- Data Source: `/Users/zhouzhou/output/country_data.py` (34 countries, ~1500 lines)
- Template: `/Users/zhouzhou/output/html_template.py` (Chart.js dark theme)
- Generator: `/Users/zhouzhou/output/generate_dashboards.py`
- Deep Research: `/Users/zhouzhou/research/deep_finance_*.md` (34 files, 10-25KB each)
- Cluster Research: `/Users/zhouzhou/research/cluster_*.md` (7 files)

### V1 Requirements (from 数据及看板需求V1.docx)
11 requirements implemented: CN Brand BEV Sales card, Channel Mix chart, CN BEV chart, Competitor bar chart, Finance Channels, expanded Features, Insurance consolidated, Regulator as table, per-product benchmarking, expanded OEM products

### Key Data Points Per Country
- Country overview (population, GDP, EU status, currency)
- Energy mix + Channel mix (Private/SME/RAC/Fleet)
- Brand TOP10 + BEV TOP10 + CN TOP5 + CN BEV TOP5
- Competitor model sales (27+ models with inline bar)
- Finance: base rate, loan rate, NPL, penetration, top banks, products, channels
- Insurance: players, channels, products with features
- Regulator: institution + scope + license (table format)
- Benchmarking: Tesla Model 3 vs VW ID.7 per product type (leasing/PCP/loan)
- TOP10 OEM Finance Partnerships

### Key Corrections Applied (2026-05-26)
- New Zealand OCR: 2.25%, Hong Kong: 4.00%, Denmark: 1.60%
- Philippines BSP: 4.25%, Thailand: 1.25%, Indonesia: 5.25%, UAE: 3.65%
- Sweden: No EV subsidies since Nov 2022
- Denmark: EV registration tax at 40% (2025)
- Israel: EV purchase tax rising to 70-75%
- Tesla UK insurance groups: 32-36 (not 48-50)
- VW ID.7 Germany: €54,505 (with €5,000 Kaufprämie)
