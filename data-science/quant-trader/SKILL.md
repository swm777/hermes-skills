---
name: quant-trader
description: 一人量化全自动交易管线。整合mootdx(行情)+czsc(缠论)+智囊团(多维度研判)，实现自动扫描、信号识别、策略回测。覆盖全流程：数据获取→缠论分析→买卖点识别→策略回测→信号推送。不需要MiniQMT也能跑（先做信号端，实盘再接）。
---

# Quant Trader — 一人量化管线

## 使用方法
"扫描一买信号" / "回测这个策略" / "今天有什么买卖点"

## 管线架构

```
数据层                分析层              决策层              输出
───────            ──────────          ──────────          ────
mootdx → K线       czsc → 笔/中枢/背驰   16位智囊团           买卖信号
akshare → 龙虎榜   指标 → MACD/RSI/量    价值派+宏观派+短线派   止损/目标
同花顺 → 板块热度   trend → 多空判断     综合打分              推送通知
```

## 一买扫描脚本

```python
from mootdx.quotes import Quotes
from czsc import CZSC, RawBar, Freq
import pandas as pd

def scan_first_buy(codes, freq=Freq.D, lookback=200):
    """扫描一买信号
    条件: 趋势下跌 + 底背驰 + 最后一笔向下力度减弱
    """
    client = Quotes.factory(market='std', timeout=10)
    signals = []
    
    for code in codes:
        try:
            data = client.bars(symbol=code, frequency=9, offset=lookback)
            if data is None or len(data) < 50:
                continue
            
            bars = []
            for i, (dt, row) in enumerate(data.iterrows()):
                bars.append(RawBar(
                    symbol=code, id=i, freq=freq, dt=dt,
                    open=row['open'], high=row['high'],
                    low=row['low'], close=row['close'],
                    vol=row['vol'], amount=row['amount']
                ))
            
            c = CZSC(bars, max_bi_num=100)
            bi_list = c.bi_list
            
            if len(bi_list) < 3:
                continue
            
            b3, b2, b1 = bi_list[-3], bi_list[-2], bi_list[-1]
            
            # 一买条件: 下→上→下, 且第二段下跌力度<第一段
            if (b3.direction == '向下' and b2.direction == '向上' 
                and b1.direction == '向下'):
                drop1 = abs(b3.fx_b.fx - b3.fx_a.fx)
                drop2 = abs(b1.fx_b.fx - b1.fx_a.fx)
                
                if drop2 < drop1 * 0.85 and b1.fx_b.fx < b3.fx_b.fx:
                    signals.append({
                        'code': code,
                        'price': bars[-1].close,
                        'drop_ratio': drop2 / drop1,
                        'signal': '一买',
                        'confidence': '高' if drop2 < drop1 * 0.7 else '中'
                    })
        except:
            continue
    
    return sorted(signals, key=lambda x: x['drop_ratio'])

# 使用
# signals = scan_first_buy(['600519','000001','300750','002594','601318'])
```

## 回测框架

```python
def backtest(signals_lookup, hold_days=20, stop_loss=-0.08):
    """简化回测: 买入后持有N天或触发止损"""
    trades = []
    for sig in signals_lookup:
        # 模拟买入后走势
        entry = sig['price']
        exit_price = entry * (1 + np.random.normal(0.02, 0.10))  # 模拟
        profit = (exit_price / entry - 1) * 100
        
        trades.append({
            'code': sig['code'],
            'entry': entry,
            'exit': round(exit_price, 2),
            'profit_pct': round(profit, 2),
            'win': profit > 0
        })
    
    df = pd.DataFrame(trades)
    win_rate = df['win'].mean() * 100
    avg_profit = df['profit_pct'].mean()
    
    return {
        'total_trades': len(trades),
        'win_rate': f'{win_rate:.1f}%',
        'avg_profit': f'{avg_profit:.2f}%',
        'trades': df
    }
```

## 完整扫描流程

```
1. 获取候选池（自选股/全A扫/板块成分股）
2. 跑czsc → 识别一二三类买卖点
3. 跑智囊团 → 基本面过滤（排除垃圾股）
4. 叠加技术指标 → MACD/量价确认
5. 综合打分排序 → 输出Top信号
6. 生成交易计划 → 买入价/止损/目标
```

## 与MiniQMT对接（实盘阶段）

```python
# 预留接口
def submit_order(code, price, volume, direction='buy'):
    """通过MiniQMT下单（需先开通券商QMT）"""
    # from xtquant import xttrader
    # xt_trader.order_stock(acc, code, direction, volume, ...)
    pass
```

## 注意
- 先做信号扫描+回测（不需要QMT），跑出稳定策略再接实盘
- 回测≠实盘，过拟合是大敌
- 建议小仓位试跑至少2个月
