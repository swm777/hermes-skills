---
name: model-builder
description: DCF估值模型构建器。输入股票代码，自动拉取财务数据，搭建自由现金流折现模型，输出目标价、敏感性分析和蒙特卡洛模拟。适用于个股估值、投资决策辅助。
---

# Model Builder — DCF估值模型

## 使用方法
"给茅台做个DCF估值" / "600519现在贵不贵" / "算一下宁德时代值多少钱"

## 模型流程
① 获取财务数据 → ② 计算FCFF → ③ 估计WACC → ④ 预测增长 → ⑤ DCF折现 → ⑥ 敏感性分析 → ⑦ 蒙特卡洛 → ⑧ 投资建议

## 核心代码

### FCFF计算
```python
def calc_fcff(ebit, tax_rate, da, capex, delta_wc):
    return ebit * (1 - tax_rate) + da - capex - delta_wc
```

### WACC估计 (CAPM)
```python
def estimate_wacc(beta=1.1):
    rf = 0.025       # 无风险利率 10年国债≈2.5%
    erp = 0.065      # 中国股权风险溢价≈6.5%
    return rf + beta * erp
```

### DCF折现
```python
def dcf_valuation(fcff_base, growth_rate, wacc, years=5, terminal_growth=0.025):
    pv_sum = 0
    fcff = fcff_base
    for y in range(1, years + 1):
        fcff *= (1 + growth_rate)
        pv_sum += fcff / ((1 + wacc) ** y)
    terminal = fcff * (1 + terminal_growth) / (wacc - terminal_growth)
    pv_terminal = terminal / ((1 + wacc) ** years)
    return pv_sum + pv_terminal
```

### 敏感性分析
```python
def sensitivity_analysis(fcff, wacc_range, growth_range, steps=6):
    results = []
    for w in np.linspace(wacc_range[0], wacc_range[1], steps):
        row = []
        for g in np.linspace(growth_range[0], growth_range[1], steps):
            row.append(round(dcf_valuation(fcff, g, w) / 1e8, 1))
        results.append(row)
    return pd.DataFrame(results,
        index=[f"WACC{w:.1%}" for w in np.linspace(wacc_range[0], wacc_range[1], steps)],
        columns=[f"g{g:.0%}" for g in np.linspace(growth_range[0], growth_range[1], steps)])
```

### 蒙特卡洛模拟
```python
def monte_carlo(fcff, wacc_m, wacc_s, growth_m, growth_s, n=1000):
    np.random.seed(42)
    evs = []
    for _ in range(n):
        w = np.clip(np.random.normal(wacc_m, wacc_s), 0.05, 0.20)
        g = np.clip(np.random.normal(growth_m, growth_s), -0.05, 0.30)
        evs.append(dcf_valuation(fcff, g, w))
    evs = np.array(evs)
    return {'mean': np.mean(evs), 'median': np.median(evs),
            'p10': np.percentile(evs, 10), 'p90': np.percentile(evs, 90)}

## 注意事项
- A股WACC建议9%-15%
- 永续增长率≤GDP增速(3%-5%)
- 金融股不适合DCF，用PB-ROE替代
- 周期股用正常化利润
