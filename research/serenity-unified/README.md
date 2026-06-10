# Serenity 供应链卡点 Skill 合集 — 综合分析项目

# Serenity Supply Chain Chokepoint Skill Collection — Unified Analysis Project

<div align="center">
  <img src="./BestSkillFromAT.png" alt="BestSkillFromAT" width="720">
</div>

---

> **🌐 Bilingual Project** | English readers: jump to [Quick Start (EN)](#quick-start-en) | 中文读者: 直接往下读

---

## 项目概述 | Project Overview

> 本项目分析了 10 个基于 **Serenity (@aleabitoreddit)** 投资方法论的 GitHub 仓库，每个仓库都从不同角度提炼和应用了"供应链卡点"投资策略，最终整合为一个统一的终极框架。

> This project analyzes 10 GitHub repositories based on the **Serenity (@aleabitoreddit)** investment methodology. Each repository refines and applies the "supply chain chokepoint" investment strategy from a different angle, ultimately integrated into one unified ultimate framework.

> **Serenity 是谁？** Reddit WSB 传奇人物，以"RISC-V+AI"框架在 2019-2024 年间实现约 3800% 收益率。核心方法论：沿着产业链向上游追溯，找到"一旦断货，万亿产业就要地震"的关键节点——那个节点上的小盘股，就是下一个暴击机会。

> **Who is Serenity?** A Reddit WSB legend who achieved ~3,800% returns from 2019-2024 using the "RISC-V+AI" framework. Core methodology: trace upstream along the supply chain to find the critical node where "if supply stops, a trillion-dollar industry shakes" — the small-cap stock sitting at that node is the next asymmetric opportunity.

---

## 仓库总览 | Repository Overview (10/10)

| # | 仓库 Repository | 特色 Highlight | 评分 Rating |
|---|-----------------|---------------|-------------|
| 1 | [muxuuu/serenity-skill](https://github.com/muxuuu/serenity-skill) | 核心方法论 + 最系统化 / Core methodology, most systematic | ⭐⭐⭐⭐⭐ |
| 2 | [lanfuli/aleabito-serenity-skills](https://github.com/lanfuli/aleabito-serenity-skills) | 推文档案库 + 注意力雷达 / Tweet archive + attention radar | ⭐⭐⭐⭐ |
| 3 | [haskaomni/serenity](https://github.com/haskaomni/serenity) | Alpha 假设 + 本地数据库 / Alpha hypothesis + local DB | ⭐⭐⭐⭐ |
| 4 | [W-Y-P/Serenity-aleabitoreddit-skill](https://github.com/W-Y-P/Serenity-aleabitoreddit-skill) | 完整框架 + 财务翻译最深 / Full framework, deepest financial translation | ⭐⭐⭐⭐⭐ |
| 5 | [ZadAnthony/serenity-skill](https://github.com/ZadAnthony/serenity-skill) | 中文 Claude 版 + 防幻觉最强 / Chinese Claude edition, strongest anti-hallucination | ⭐⭐⭐⭐⭐ |
| 6 | [fadewalk/serenity-stock-choke](https://github.com/fadewalk/serenity-stock-choke) | A 股卡脖子 + 政策适配最佳 / A-share chokepoint, best policy adaptation | ⭐⭐⭐⭐⭐ |
| 7 | [leslieyeo/serenity-reply](https://github.com/leslieyeo/serenity-reply) | 深度资料 + 心智模型蒸馏 / Deep materials + mental model distillation | ⭐⭐⭐⭐ |
| 8 | [zongmin-yu/serenity-skills](https://github.com/zongmin-yu/serenity-skills) | 半导体供应链 + 垂直领域 / Semiconductor supply chain, vertical domain | ⭐⭐⭐ |
| 9 | [yan-labs/serenity-aleabitoreddit](https://github.com/yan-labs/serenity-aleabitoreddit) | 推文档案 + 战绩校准 + skills.sh / Tweet archive + track record calibration | ⭐⭐⭐⭐⭐ |
| 10 | [xvhaoran778-cyber/Serenity.SKILL](https://github.com/xvhaoran778-cyber/Serenity.SKILL) | 跨市场卡点 + 贝叶斯更新 / Cross-market chokepoint + Bayesian updating | ⭐⭐⭐⭐ |

> 仓库 #9 此前因 SSL 问题未能获取，现已解决。仓库 #10 为 Oxagata-prog/serenity-skill (404) 的社区替代方案。
>
> Repo #9 was previously inaccessible due to SSL issues — now resolved. Repo #10 is the community replacement for Oxagata-prog/serenity-skill (404).

---

## 项目结构 | Project Structure

```
BestSerenitySkillFromAT/
├── README.md                              # 本文件 (中英双语) / This file (bilingual)
├── SKILL.md                               # v3.0 Skill 入口 (中文) / CN entry point
├── SKILL.en.md                            # v3.0 Skill entry (English)
├── FINAL_UNIFIED_SKILL.md                 # v2.0 存档版 / Archived v2.0 (single file)
├── LICENSE                                # MIT
│
├── skills/serenity-unified/               # v3.0 模块化 Skill / Modular Skill
│   ├── SKILL.md                           #   核心指令 (中文) / Core instructions (CN)
│   ├── SKILL.en.md                        #   Core instructions (English)
│   ├── knowledge/                         #   参考知识库 / Reference knowledge base
│   │   ├── market-adaptation.md           #     A 股/美股/全球市场适配 (中文)
│   │   ├── market-adaptation.en.md        #     A-share/US/Global adaptation (English)
│   │   ├── mental-models.md               #     心智模型 + 决策启发式 (英中混合)
│   │   └── supply-chain-map.md            #     供应链地图 (英中混合)
│   └── templates/                         #   报告模板 / Report templates
│       ├── single-stock-report.md         #     单股报告模板 (中文)
│       ├── single-stock-report.en.md      #     Single-stock template (English)
│       ├── sector-report.md               #     赛道报告模板 (中文)
│       └── sector-report.en.md            #     Sector report template (English)
│
├── docs/                                  # 分析文档 / Analysis docs
│   ├── ANALYSIS_SUMMARY.md                #   10 仓库深度对比 / 10-repo deep comparison
│   └── DEEP_OPTIMIZATION_REPORT.md        #   深度优化分析 / Deep optimization analysis
│
├── muxuuu-serenity-skill/                 # 仓库 1-10 (克隆的子仓库)
│   ...                                    #   Cloned sub-repos
```

---

## 核心文档 | Core Documents

- **[SKILL.md](./SKILL.md)** — v3.0 Skill 入口 (中文)，根目录，各平台一行安装
- **[SKILL.en.md](./SKILL.en.md)** — v3.0 Skill entry (English), one-line install for all platforms
- **[skills/serenity-unified/](./skills/serenity-unified/)** — 模块化结构 (核心指令 + knowledge/ + templates/)
- **[FINAL_UNIFIED_SKILL.md](./FINAL_UNIFIED_SKILL.md)** — v2.0 存档版 (单文件 691 行)
- **[docs/ANALYSIS_SUMMARY.md](./docs/ANALYSIS_SUMMARY.md)** — 10 仓库逐一深度分析、核心差异矩阵
- **[docs/DEEP_OPTIMIZATION_REPORT.md](./docs/DEEP_OPTIMIZATION_REPORT.md)** — 演化路径、防幻觉机制等独立分析

---

## 快速开始 | Quick Start

### 一行安装 | One-Line Install

根据你的 Agent 工具，选一条命令执行即可。Pick the command for your agent:

```bash
# Codex — 告诉 Codex / Tell Codex:
"帮我从 GitHub 拉取 yux1azhengye/BestSerenitySkillFromAT 并把根目录的 SKILL.md 安装为 skill"

# skills.sh (Codex 通用 / universal)
npx skills add yux1azhengye/BestSerenitySkillFromAT

# Claude Code (全局，任意目录可用 / global, works from any directory)
git clone https://github.com/yux1azhengye/BestSerenitySkillFromAT.git ~/.claude/skills/serenity-unified-skill

# Cursor
git clone https://github.com/yux1azhengye/BestSerenitySkillFromAT.git /tmp/serenity && cp /tmp/serenity/SKILL.md .cursor/rules/serenity-unified.mdc

# Gemini CLI
git clone https://github.com/yux1azhengye/BestSerenitySkillFromAT.git /tmp/serenity && cp /tmp/serenity/SKILL.md GEMINI.md
```

### 触发示例 | Trigger Examples

```
# 中文 / Chinese
/serenity 分析一下 $NVDA 这只票值不值得
帮我用 Serenity 的方式看 1.6T 光模块这条链还有没有没被定价的卡点
用 Serenity 的方式看 A 股 AI 半导体哪个最值得研究

# English
/serenity analyze whether $NVDA is worth investing
Use Serenity's method to see if there are unpriced chokepoints in the 1.6T optical chain
Find unknown bottlenecks in the AI infrastructure supply chain
Rank the best chokepoint candidates in US semiconductors
```

### English Users: Quick Start (EN)

For English readers, use the English skill files:

```bash
# Claude Code (English skill)
git clone https://github.com/yux1azhengye/BestSerenitySkillFromAT.git ~/.claude/skills/serenity-unified-skill
# Then tell Claude: "Use the serenity-unified skill"

# Or manually load the English skill:
cp ~/.claude/skills/serenity-unified-skill/SKILL.en.md ~/.claude/skills/serenity-unified-skill/SKILL.md
```

The English skill is in `SKILL.en.md` (root) and `skills/serenity-unified/SKILL.en.md`. All reference files have `.en.md` English versions in `knowledge/` and `templates/`.

---

### 按需选择原始仓库 | Pick an Original Repo by Need

- **A 股用户 / A-share investors**：`fadewalk/serenity-stock-choke` 或 `ZadAnthony/serenity-skill`
- **美股用户 / US stock investors**：`W-Y-P/Serenity-aleabitoreddit-skill` 或 `muxuuu/serenity-skill`
- **追踪 Serenity 本人 / Track Serenity**：`lanfuli/aleabito-serenity-skills` (6,120 帖/posts + 注意力雷达/attention radar)
- **本地可视化 / Local visualization**：`haskaomni/serenity` (`python3 scripts/server.py --port 8787`)

### 克隆全部仓库 | Clone All Repos

```bash
git clone https://github.com/muxuuu/serenity-skill.git muxuuu-serenity-skill
git clone https://github.com/lanfuli/aleabito-serenity-skills.git lanfuli-aleabito-serenity-skills
git clone https://github.com/haskaomni/serenity.git haskaomni-serenity
git clone https://github.com/W-Y-P/Serenity-aleabitoreddit-skill.git WYP-Serenity-aleabitoreddit-skill
git clone https://github.com/ZadAnthony/serenity-skill.git ZadAnthony-serenity-skill
git clone https://github.com/fadewalk/serenity-stock-choke.git fadewalk-serenity-stock-choke
git clone https://github.com/leslieyeo/serenity-reply.git leslieyeo-serenity-reply
git clone https://github.com/zongmin-yu/serenity-skills.git zongmin-yu-serenity-skills
git clone https://github.com/yan-labs/serenity-aleabitoreddit.git yan-labs-serenity-aleabitoreddit
git clone https://github.com/xvhaoran778-cyber/Serenity.SKILL.git xvhaoran778-cyber-Serenity-SKILL
```

---

## 免责声明 | Disclaimer

> **不是投资建议 (NOT investment advice)**。本项目仅用于学习、研究投资分析方法论。任何标的、信念分档、估值区间都不构成买卖建议；真金白银请自行 DYOR。

> **NOT investment advice.** This project is for learning and studying investment analysis methodology only. No ticker, conviction tier, or valuation range constitutes buy/sell advice; do your own research before committing real capital.

> **第三方独立提炼，非官方、未获授权背书**。本项目基于多个公开仓库自底向上整合，与 @aleabitoreddit 没有任何关联。

> **Third-party independent synthesis, unofficial and unauthorized.** This project is bottom-up integration from multiple public repositories and has no affiliation with @aleabitoreddit.

> **他的战绩均为其本人自述、未经独立审计**，引用时务必带此限定。投资有风险，决策风险与后果由使用者自行承担。

> **All track records are self-reported and unaudited.** Always cite with this qualifier. Investing involves risk; users bear full responsibility for their decisions and consequences.

---

## 许可证 | License

MIT — 随便用，随便改，随便造。方法论提炼内容供个人学习/研究使用，底层投资观点版权归原作者所有。

MIT — use freely, modify freely, build freely. Methodology synthesis content is for personal learning/research use; underlying investment views are copyright their original authors.

---

## 致谢 | Acknowledgments

感谢以下开源作者 / Thanks to the following open-source authors：@muxuuu、@ZadAnthony、@W-Y-P、@fadewalk、@lanfuli、@haskaomni、@leslieyeo、@zongmin-yu、@yan-labs、@xvhaoran778-cyber。

以及原始方法论来源 / And the original methodology source：**Serenity (@aleabitoreddit)** — X/Twitter 上的 AI/半导体供应链分析师 / AI & semiconductor supply chain analyst on X/Twitter.

---

*本文档于 2026 年 6 月 10 日更新 | Last updated: June 10, 2026*
