---
name: awesome-finance
description: A股每日情绪日报。聚合同花顺板块、涨停板、龙虎榜、全球快讯、指数行情，生成一份结构化的市场情绪报告。适合每日盘前/盘后快速了解市场全貌。
---

# Awesome Finance — A股情绪日报

## 概述

一键生成当日A股市场情绪报告，覆盖：指数行情、板块热点、涨停分析、龙虎榜、财经快讯。数据源：同花顺 + 东方财富 + akshare。

## 使用方法

让AI执行：
- "生成今天A股情绪日报"
- "今天市场怎么样"
- "盘前简报"

## 数据流程

```
① 指数行情 → 上证/深证/创业板涨跌
② 同花顺板块 → 涨跌TOP5 + 主力资金流向
③ 涨停板统计 → 涨停数量 + 连板龙头
④ 龙虎榜 → 机构动向 + 游资席位
⑤ 全球快讯 → 重要财经新闻摘要
⑥ 情绪评分 → 综合打分(1-10)
```

## 核心代码

### 基础依赖

```python
import akshare as ak
import requests, random
import pandas as pd
from datetime import datetime

today = datetime.now().strftime("%Y%m%d")
```

### ① 指数行情（首选：腾讯财经 · 不封IP · 秒回）

```python
def get_index_tencent():
    """腾讯财经 — 三大指数实时行情（推荐，秒回不封IP）
    PB在索引46, PE在索引39, 涨跌幅在索引32
    """
    codes = ['sh000001','sz399001','sz399006']
    names = {'sh000001':'上证指数','sz399001':'深证成指','sz399006':'创业板指'}
    r = requests.get("https://qt.gtimg.cn/q=" + ",".join(codes), timeout=5)
    r.encoding = 'gbk'
    
    results = []
    for line in r.text.strip().split('\n'):
        if '="' not in line: continue
        code = line.split('="')[0].replace('v_','')
        f = line.split('="')[1].split('~')
        results.append({
            'name': names.get(code, f[1]),
            'price': float(f[3]) if f[3] else 0,
            'change_pct': float(f[32]) if f[32] else 0,
        })
    return results

# 备用: akshare（经常超时，不推荐）
# df = ak.stock_zh_index_spot_em()
```

### ② 同花顺板块热点

```python
def get_sector_hotspots():
    """获取同花顺板块涨跌排行"""
    headers = {
        "Host": "dq.10jqka.com.cn",
        "Content-Type": "application/json",
        "User-Agent": f"Mozilla/5.0 ... userid/{random.randint(710000000, 712680385)}",
        "Origin": "https://localhost:8088",
        "Referer": "https://localhost:8088/",
    }
    payload = {
        "sort_info": {"sort_field": "0", "sort_type": "desc"},
        "history_info": {"history_type": "0", "end_date": f"{today}150000", "start_date": f"{today}093000"},
        "type": 0,
        "page_info": {"page_size": 100, "page": 1}
    }
    r = requests.post(
        "https://dq.10jqka.com.cn/interval_calculation/block_info/v1/get_block_list",
        json=payload, headers=headers, timeout=10
    )
    items = r.json()["data"]["list"]
    
    # 涨幅TOP5
    top5 = sorted(items, key=lambda x: x.get('margin_of_increase', 0), reverse=True)[:5]
    # 跌幅TOP5
    bottom5 = sorted(items, key=lambda x: x.get('margin_of_increase', 0))[:5]
    # 主力净流入TOP5
    inflow5 = sorted(items, key=lambda x: x.get('net_inflow_of_main_force', 0), reverse=True)[:5]
    # 主力净流出TOP5
    outflow5 = sorted(items, key=lambda x: x.get('net_inflow_of_main_force', 0))[:5]
    
    return {
        'top_gainers': top5,
        'top_losers': bottom5,
        'top_inflow': inflow5,
        'top_outflow': outflow5,
        'total_sectors': len(items)
    }
```

### ③ 涨停板统计

```python
def get_limit_up_summary():
    """涨停板概况"""
    try:
        df = ak.stock_zt_pool_em(date=today)
        # 统计行业分布
        return {
            'total': len(df),
            'market': '活跃' if len(df) > 80 else ('温和' if len(df) > 40 else '冷清')
        }
    except:
        return {'total': 0, 'market': '无数据'}
```

### ④ 龙虎榜动向

```python
def get_dragon_tiger_summary():
    """龙虎榜资金动向"""
    try:
        df = ak.stock_lhb_detail_em(start_date=today, end_date=today)
        if len(df) == 0:
            return {'count': 0, 'message': '今日无龙虎榜数据'}
        
        # 净买入TOP5
        df['净买额'] = pd.to_numeric(df['龙虎榜净买额'], errors='coerce')
        top_buy = df.nlargest(5, '净买额')
        
        # 机构动向统计
        inst_buy = df[df['解读'].str.contains('机构买入', na=False)]
        
        return {
            'count': len(df),
            'top_buy': top_buy[['代码','名称','净买额','解读']].to_dict('records'),
            'inst_count': len(inst_buy),
            'message': f"机构净买入{len(inst_buy)}只"
        }
    except Exception as e:
        return {'count': 0, 'message': str(e)[:100]}
```

### ⑤ 全球财经快讯

```python
def get_news_highlights():
    """重要财经快讯摘要"""
    try:
        df = ak.stock_info_global_em()
        recent = df.head(20)
        return recent[['title','content']].to_dict('records') if 'content' in df.columns else [{'title': t} for t in recent['title'].tolist()]
    except:
        return []
```

### ⑥ 情绪评分

```python
def calc_sentiment(limit_up, sectors, lhb):
    """综合情绪评分 1-10"""
    score = 5  # 基准
    
    # 涨停数量
    zt = limit_up.get('total', 0)
    if zt > 100: score += 2
    elif zt > 60: score += 1
    elif zt < 30: score -= 1
    
    # 板块涨跌比
    # (简化: 用top_gainer涨跌幅判断)
    
    # 龙虎榜机构动向
    inst = lhb.get('inst_count', 0)
    if inst > 10: score += 1
    elif inst < 3: score -= 1
    
    return max(1, min(10, score))
```

## 报告模板

执行时按以下模板输出：

```markdown
📊 A股情绪日报 | {日期}

━━━━ 指数行情 ━━━━
{三大指数表格}

━━━━ 板块热点 ━━━━
🔥 涨幅TOP5: ...
📉 跌幅TOP5: ...
💰 主力流入TOP5: ...

━━━━ 涨停分析 ━━━━
涨停 {N} 只 | 市场情绪: {描述}

━━━━ 龙虎榜 ━━━━
上榜 {N} 只 | {机构动向}
净买入TOP3: ...

━━━━ 财经快讯 ━━━━
· 摘要1
· 摘要2
· 摘要3

━━━━ 情绪评分 ━━━━
{分数}/10 | {解读}
```
