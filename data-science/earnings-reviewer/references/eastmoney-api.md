# 东方财富数据中心 API 参数手册

> 发现日期: 2026-06-09 · 用于替代不稳定的akshare财报接口

## 基础URL

```
https://datacenter-web.eastmoney.com/api/data/v1/get
```

## 已验证的报表名 (reportName)

| reportName | 内容 | 关键字段 |
|---|---|---|
| `RPT_DMSK_FN_INCOME` | 利润表 | TOTAL_OPERATE_INCOME, PARENT_NETPROFIT, OPERATE_PROFIT, SALE/MANAGE/FINANCE_EXPENSE |
| `RPT_DMSK_FN_BALANCE` | 资产负债表 | (需验证字段) |
| `RPT_DMSK_FN_CASHFLOW` | 现金流量表 | (需验证字段) |

### ❌ 无效报表名
- `RPT_DMSK_FN_MAININDICATOR` — 返回 "报表配置不存在"
- `RPT_LICO_FN_CPD` — 返回 "REPORT_DATE返回字段不存在"
- `RPT_F10_FINANCE_MAINFINADATA` — 字段不匹配

## 标准请求参数

```python
params = {
    "reportName": "RPT_DMSK_FN_INCOME",
    "columns": "ALL",                          # 或指定字段逗号分隔
    "filter": '(SECURITY_CODE="300604")',       # 6位数字代码
    "pageNumber": 1,
    "pageSize": 5,
    "sortTypes": -1,                            # -1=降序(最新在前)
    "sortColumns": "REPORT_DATE",
}
headers = {
    "User-Agent": "Mozilla/5.0",
    "Referer": "https://data.eastmoney.com/",   # 必须带Referer
}
```

## 利润表关键字段映射

| 东财字段 | 中文含义 | 单位 |
|---|---|---|
| REPORT_DATE | 报告期 | 日期 |
| TOTAL_OPERATE_INCOME | 营业总收入 | 元 |
| TOTAL_OPERATE_COST | 营业总成本 | 元 |
| OPERATE_PROFIT | 营业利润 | 元 |
| PARENT_NETPROFIT | 归母净利润 | 元 |
| SALE_EXPENSE | 销售费用 | 元 |
| MANAGE_EXPENSE | 管理费用 | 元 |
| FINANCE_EXPENSE | 财务费用 | 元 |
| INCOME_TAX | 所得税 | 元 |
| SECURITY_NAME_ABBR | 股票简称 | — |
| DATE_TYPE_CODE | 报表类型 | 003=季报 |

## 数据使用要点

1. 所有金额字段单位为**元**，需除以1e8转为亿
2. `filter`支持多条件：`(SECURITY_CODE="300604")(REPORT_DATE>"2025-01-01")`
3. 请求间隔建议≥1秒，带随机抖动
4. 必须带 `Referer: https://data.eastmoney.com/`，否则可能被拒

## 与其他数据源对比

| 数据源 | 财报稳定性 | 响应速度 |
|---|---|---|
| 东财直连(datacenter-web) | ✅ 极稳 | 0.4s |
| akshare(封装层) | ❌ 常返回None | 2-5s |
| 新浪(quotes.sina.cn) | ❌ 接口已废 | N/A |
| 同花顺(data.10jqka.com.cn) | ⚠️ HTML解析脆弱 | 0.4s |
