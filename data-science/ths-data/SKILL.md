---
name: ths-data
description: 同花顺+东财 A股数据获取。覆盖行业板块行情、板块成分股、龙虎榜、涨停板池、研报、财务数据。当需要获取A股实时行情、龙虎榜、涨停分析、板块热点、研报检索时使用。
---

# 同花顺 & 东方财富 A股数据接口

## 概述

直连同花顺公开HTTP接口 + akshare（东方财富数据），覆盖：行业板块、涨停板、龙虎榜、研报、K线、财务。**全部免费，无需注册。**

## 数据源说明

| 数据 | 来源 | 方法 | 优先级 |
|---|---|---|---|
| **K线/盘口** | 通达信(永不封IP) | `mootdx.Quotes().bars()` | 🔑 首选 |
| 行业板块行情 | 同花顺 dq.10jqka.com.cn | HTTP POST | ⭐ |
| 板块成分股 | 同花顺 dq.10jqka.com.cn | HTTP POST | ⭐ |
| 涨停板池 | 东方财富（akshare） | `ak.stock_zt_pool_em()` | |
| 龙虎榜 | 东方财富（akshare） | `ak.stock_lhb_detail_em()` | |
| 研报 | 东方财富（akshare） | `ak.stock_research_report_em()` | |
| PE/PB/市值 | 腾讯财经(不封IP) | HTTP | 🔑 |
| 财务 | 新浪/东财 | akshare/mootdx | |

## 核心代码

### 0. mootdx K线（永不封IP · 首选）

```python
from mootdx.quotes import Quotes
client = Quotes.factory(market='std', timeout=10)

# 频率: 9=日线, 5=周线, 6=月线, 2=30分钟, 3=60分钟
df = client.bars(symbol='600519', frequency=9, offset=100)
# 返回DataFrame, DatetimeIndex, columns: open/close/high/low/vol/amount

# 喂给 czsc:
from czsc import CZSC, RawBar, Freq
bars = []
for i, (dt, row) in enumerate(df.iterrows()):
    bars.append(RawBar(symbol=sym, id=i, freq=Freq.D, dt=dt,
        open=row['open'], high=row['high'], low=row['low'],
        close=row['close'], vol=row['vol'], amount=row['amount']))
c = CZSC(bars, max_bi_num=300)
```

### 1. 同花顺行业板块行情

```python
import requests, random

headers = {
    "Host": "dq.10jqka.com.cn",
    "Accept": "application/json, text/plain, */*",
    "Content-Type": "application/json",
    "Origin": "https://localhost:8088",
    "User-Agent": f"Mozilla/5.0 (iPhone; CPU iPhone OS 17_2_1 like Mac OS X) AppleWebKit/605.1.15 userid/{random.randint(710000000, 712680385)}",
    "Referer": "https://localhost:8088/",
}

def get_ths_blocks(date="20260602", page_size=50):
    """获取同花顺行业/概念/地域板块行情
    date: YYYYMMDD格式，数据从2022-08-22起
    返回: [{block_name, block_code, margin_of_increase, net_inflow_of_main_force, turnover}]
    """
    payload = {
        "sort_info": {"sort_field": "0", "sort_type": "desc"},
        "history_info": {"history_type": "0", "end_date": f"{date}150000", "start_date": f"{date}093000"},
        "type": 0,
        "page_info": {"page_size": page_size, "page": 1}
    }
    r = requests.post(
        "https://dq.10jqka.com.cn/interval_calculation/block_info/v1/get_block_list",
        json=payload, headers=headers, timeout=10
    )
    data = r.json()
    return data["data"]["list"], data["data"]["total"]
```

### 2. 同花顺板块成分股

```python
def get_ths_block_stocks(block_code, date="20260602"):
    """获取同花顺板块成分股及当日表现
    block_code: 如 '881270'（元件）
    返回: [{stock_code, stock_name, margin_of_increase, net_inflow_of_main_force, amount}]
    """
    payload = {
        "block_code": block_code,
        "sort_info": {"sort_field": "0", "sort_type": "desc"},
        "history_info": {"history_type": "1", "end_date": f"{date}000000", "start_date": f"{date}000000"},
        "page_info": {"page_size": 100, "page": 1},
        "block_market": "48"
    }
    r = requests.post(
        "https://dq.10jqka.com.cn/interval_calculation/stock_info/v1/get_stock_list_by_block",
        json=payload, headers=headers, timeout=10
    )
    data = r.json()
    return data["data"]["list"], data["data"]["total"]
```

### 3. 涨停板池

```python
import akshare as ak

def get_limit_up_pool(date="20260605"):
    """获取当日涨停板股票池
    date: YYYYMMDD
    返回: DataFrame含代码、名称、涨跌幅、封单等
    """
    return ak.stock_zt_pool_em(date=date)
```

### 4. 龙虎榜

```python
def get_dragon_tiger(start_date="20260530", end_date="20260605"):
    """获取龙虎榜数据
    返回: DataFrame含代码、名称、上榜日、净买额、解读、上榜原因
    """
    return ak.stock_lhb_detail_em(start_date=start_date, end_date=end_date)
```

### 5. 研报

```python
def get_research_reports(stock_code="600519"):
    """获取个股研报
    stock_code: 6位代码
    """
    return ak.stock_research_report_em(symbol=stock_code)
```

## 使用示例

### 分析今日热点板块

```python
blocks, total = get_ths_blocks(date="20260605")
# 按涨跌幅排序，取Top10
blocks_sorted = sorted(blocks, key=lambda x: x.get("margin_of_increase", 0), reverse=True)
for b in blocks_sorted[:10]:
    print(f"{b['block_name']}: {b['margin_of_increase']:.2f}% 主力净流入:{b['net_inflow_of_main_force']/1e8:.1f}亿")
```

### 一键获取涨停+龙虎榜

```python
# 今日涨停
zt = get_limit_up_pool("20260605")
print(f"涨停 {len(zt)} 只")

# 近5天龙虎榜
lhb = get_dragon_tiger(start_date="20260601", end_date="20260605")
print(f"龙虎榜 {len(lhb)} 条")
```

## 注意事项

1. **同花顺接口是公开HTTP接口，非官方API**——随时可能变更，频率过高会被封IP
2. 同花顺板块数据最早从 2022-08-22 开始
3. `userid` 随机化避免被识别为爬虫
4. akshare 数据源为东方财富，非同花顺，但数据维度互补
5. K线数据优先用 mootdx/腾讯（不封IP），避免频繁请求东财

## 相关工具

- **mootdx**: K线+盘口+财务快照（通达信协议，不封IP）
- **腾讯财经**: PE/PB/市值/换手率（HTTP，不封IP）
- **czsc**: 缠论技术分析（已安装0.10.12）
- **czsc-thinking**: 缠论思维框架Skill
