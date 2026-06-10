---
name: earnings-reviewer
description: 财报审阅员。读取个股财报，自动对比同比/环比变化，标注异常科目，计算核心财务指标（ROE/毛利率/净利率/现金流），生成一份简洁的"该不该持有"判断报告。
---

# Earnings Reviewer — 财报审阅员

## 使用方法
"看看茅台最新财报" / "600519季报点评" / "宁德时代财务健康吗"

> 📖 东财API详细参数: [references/eastmoney-api.md](references/eastmoney-api.md)

## 分析流程
① 拉取三张表 → ② 同比/环比对比 → ③ 标注异常变动(>20%) → ④ 计算核心指标 → ⑤ 综合判断

## 核心指标
- 营收增速 / 利润增速 / 毛利率 / 净利率
- ROE / ROA / 经营现金流/净利润比
- 应收账款/营收比 / 存货周转 / 资产负债率

## 核心代码

### 数据源优先级

| 优先级 | 方案 | 稳定性 | 说明 |
|--------|------|--------|------|
| 🥇 | **东财直连** | ✅ 极稳 | 绕过akshare，直接调`datacenter-web.eastmoney.com` |
| 🥈 | akshare | ⚠️ 不稳定 | 利润表/现金流经常返回None |
| ❌ | 新浪财报API | ❌ 已废 | `quotes.sina.cn`接口返回"Invalid service name" |

### 获取财报（东财直连 · 推荐）

```python
import requests

def get_income_statement(code, pagesize=5):
    """利润表 — 东财数据中心直连
    返回最近N期数据，含: 营收、净利润、营业利润、各项费用
    """
    url = "https://datacenter-web.eastmoney.com/api/data/v1/get"
    params = {
        "reportName": "RPT_DMSK_FN_INCOME",
        "columns": "ALL",
        "filter": f'(SECURITY_CODE="{code}")',
        "pageNumber": 1, "pageSize": pagesize,
        "sortTypes": -1, "sortColumns": "REPORT_DATE",
    }
    r = requests.get(url, params=params,
        headers={"User-Agent": "Mozilla/5.0", "Referer": "https://data.eastmoney.com/"},
        timeout=10)
    return r.json()["result"]["data"]

# 关键字段映射:
# TOTAL_OPERATE_INCOME → 营业总收入
# TOTAL_OPERATE_COST → 营业总成本
# OPERATE_PROFIT → 营业利润
# PARENT_NETPROFIT → 归母净利润
# SALE_EXPENSE / MANAGE_EXPENSE / FINANCE_EXPENSE → 三费
# 注意: 数据单位为元，需除以1e8转亿
```

### 同比环比分析（修订：基于东财数据）

```python
def compare_quarters(data):
    """从东财数据计算同比环比"""
    if len(data) < 2:
        return None
    
    latest = data[0]  # 最新一期
    prev_q = data[1]  # 上季度
    
    # 找去年同期（4期前，如有）
    yoy = data[4] if len(data) > 4 else data[-1]
    
    rev_latest = latest.get('TOTAL_OPERATE_INCOME', 0) or 0
    rev_yoy = yoy.get('TOTAL_OPERATE_INCOME', 0) or 0
    
    profit_latest = latest.get('PARENT_NETPROFIT', 0) or 0
    profit_yoy = yoy.get('PARENT_NETPROFIT', 0) or 0
    
    return {
        'report_date': latest.get('REPORT_DATE','')[:10],
        'revenue': rev_latest/1e8,
        'revenue_yoy': (rev_latest/rev_yoy - 1)*100 if rev_yoy else None,
        'net_profit': profit_latest/1e8,
        'profit_yoy': (profit_latest/profit_yoy - 1)*100 if profit_yoy else None,
        'gross_margin': (1 - (latest.get('TOTAL_OPERATE_COST',0) or 0)/rev_latest)*100 if rev_latest else None,
        'net_margin': (profit_latest/rev_latest)*100 if rev_latest else None,
    }
```

### 同比环比分析
```python
def compare_periods(df, periods=[-1, -4]):
    """对比最新季度与去年同期/上季度"""
    latest = df.iloc[:, 0]  # 最新一期
    results = {}
    for offset in periods:
        prev = df.iloc[:, offset]
        change = (latest - prev) / prev.abs() * 100
        results[f'vs_{offset}'] = change
    return results
```

### 异常标注
```python
def flag_anomalies(changes, threshold=20):
    """标注变动>20%的科目"""
    warnings = []
    for col, pct in changes.items():
        if abs(pct) > threshold:
            direction = '↑↑' if pct > 0 else '↓↓'
            warnings.append(f"{direction} {col}: {pct:+.1f}%")
    return warnings
```

### 关键指标计算（修订：东财数据版）

```python
def calc_key_metrics(data):
    """从东财利润表数据计算核心指标"""
    latest = data[0]
    rev = latest.get('TOTAL_OPERATE_INCOME', 0) or 0
    cost = latest.get('TOTAL_OPERATE_COST', 0) or 0
    profit = latest.get('PARENT_NETPROFIT', 0) or 0
    
    return {
        '报告期': latest.get('REPORT_DATE','')[:10],
        '营收(亿)': round(rev/1e8, 2),
        '净利润(亿)': round(profit/1e8, 2),
        '毛利率%': round((1 - cost/rev)*100, 1) if rev else 0,
        '净利率%': round(profit/rev*100, 1) if rev else 0,
        '销售费用率%': round((latest.get('SALE_EXPENSE',0) or 0)/rev*100, 1) if rev else 0,
        '管理费用率%': round((latest.get('MANAGE_EXPENSE',0) or 0)/rev*100, 1) if rev else 0,
    }
```

## 判断逻辑
```
✅ 值得持有:
  营收增速>10% + 利润增速>营收增速 + ROE>15% + 经营现金流/净利润>1

⚠️ 谨慎持有:
  营收增速>0但<10% 或 利润增速<营收增速 或 经营现金流为负

❌ 建议卖出:
  营收负增长 或 利润大幅下滑 或 ROE<5% 或 现金流持续为负
```

## 输出模板
```markdown
📋 财报审阅 | {股票}({代码}) | {报告期}

━━━━ 核心指标 ━━━━
营收 {X}亿 (同比{Y}%) | 净利润 {Z}亿 (同比{W}%)
毛利率 {M}% | ROE {R}% | 经营现金流/净利润 {C}

━━━━ 异常变动 ━━━━
{flag列表}

━━━━ 综合评价 ━━━━
{结论 + 建议}
```
