# POC03-A 最终研究记录（中文）

## 利率压力 × 市场韧性：跨资产压力能否预测未来股市下行风险？

**研究状态：已结束（CLOSED）**

**最终结论：Primary Hypothesis --- NOT SUPPORTED /
原始假设未获得样本外支持**

------------------------------------------------------------------------

## 1. 研究目的

POC03-A 想回答的问题是：

> 当美国国债利率压力很大，同时市场自身的"承压能力"开始恶化时，未来一个月股市出现较大下跌的风险是否会增加？

这个项目不是为了预测"明天 SPY 涨还是跌"，而是服务于更实际的投资目标：

**风险控制 / Risk Sizing / 尽量避开大幅回撤。**

我们希望找到一种能够帮助判断"什么时候应该更谨慎"的跨资产风险信号。

------------------------------------------------------------------------

## 2. 原始假设

研究开始时冻结的方向性假设是：

**High Rates Pressure + Weakening Market Resilience → More Negative
FAE_20D**

即：

> 高利率压力 + 市场韧性下降 → 未来 20 个交易日出现更严重的下行。

核心标签为：

### FAE_20D --- 20-Day Forward Adverse Excursion

定义为从今天开始，未来 20 个交易日中 SPY 相对于今天出现过的最低收益率：

`FAE_20D(t) = min(SPY(t+k) / SPY(t) - 1), k = 1,...,20`

FAE 越负，表示未来经历的下行越严重。

它和普通的 20 日 forward return 不一样。例如 SPY 从 100 跌到
85，之后又涨回 101，20 日收益可能是 +1%，但投资者中间实际上经历了 -15%
的下行。FAE 能把这个风险保留下来。

20 个交易日的 horizon 在 outcome analysis
前冻结，没有因为后面的结果不好而更换。

------------------------------------------------------------------------

## 3. 数据与样本划分

最终完整分析样本：

-   5,380 个完整 observations
-   2005-01-05 至 2026-09-03

研究时期：

-   **DEV：2005-01-05 至 2022-12-30**
-   **CONFIRM：2023-01-03 至 2025-12-31**
-   **LIVE：2026-01-02 以后**

2026 年只作为 live case，而不是完全 untouched OOS，因为 2026
年当时的市场现象本身参与了最初研究问题的形成。

不同数据源先在各自 native calendar 上计算 rolling
features，再进行跨资产对齐。

没有使用 forward fill 去人为填补缺失的宏观数据。

------------------------------------------------------------------------

## 4. 最终冻结的 13 个 Features

### Rates Pressure --- 利率压力

1.  `LevelZ252`：当前 10Y Treasury yield 相对于过去 252 个 observations
    的高度。
2.  `Momentum20BP`：最近 20 个 observations 的 10Y yield 变化，单位 bp。
3.  `ShockZ60`：当天利率变化相对于过去 60 个 observations 是否异常。
4.  `DistanceHighVolAdj60`：经过波动率调整后，当前 10Y yield 距离 60
    期高点有多远。

这四个变量分别对应：

-   Level = 海拔
-   Momentum = 最近一个月爬了多少
-   Shock = 今天是否突然猛踩油门/刹车
-   Distance from High = 离山顶多远

研究过程中明确认识到：

**Level ≠ Momentum ≠ Shock ≠ Distance from High。**

### Equity Price / Concentration --- 股票价格与集中度

5.  `SPYMomentum20`
6.  `SPYDistanceHigh60`
7.  `EqualWeightRelative20`

其中 `EqualWeightRelative20` 衡量 RSP 相对 SPY 的 20
日表现，主要作为市场集中度 proxy，而不是完整 breadth 指标。

### Credit --- 信用市场

8.  `HYOASLevel`
9.  `HYOASChange20BP`

分别观察 HY OAS 的绝对水平以及最近 20 期的变化方向。

### Volatility --- 波动率

10. `VIXLevel`
11. `VIXChange20`

### FX / Cross-Asset

12. `DXYChange20`
13. `USDJPYChange20`

SPY 本身只用于构造未来 outcome label，不作为预测 feature。

------------------------------------------------------------------------

## 5. Rates Pressure 的结果

四个预先冻结的 Treasury-rate features 都没有表现出稳定、单调的：

**利率压力越大 → 未来 FAE 越差**

这种关系。

其中 `Momentum20BP` 出现了比较明显的两端效应：

-   利率快速上涨时，未来 downside 较差；
-   但利率快速下跌时，未来 downside 也较差。

因此一个可能的解释是：

> 大幅利率变化更像"不稳定市场环境"的特征，而不一定是简单的"利率上涨导致股票下跌"。

这只是后续可以研究的新假设，POC03-A 没有通过重新构造 `abs(Momentum)`
等方式去挽救原始假设。

------------------------------------------------------------------------

## 6. DEV 中 Market Response 出现了强关系

### SPY 自身价格状态

DEV 中，弱 `SPYMomentum20` 和较差的 `SPYDistanceHigh60`
都对应更严重的未来 FAE。

但这里存在一个重要区别：

> **Early Warning ≠ Damage Confirmation。**

如果 SPY
已经明显跌破趋势，模型再告诉我们未来风险较高，它可能主要是在识别一个已经发生的
stress regime，而不是提前预警。

### Credit

`HYOASChange20BP` 在 DEV 中非常明显。

HY OAS widening 最严重的 Q5：

-   Mean FAE_20D ≈ **-4.24%**
-   Median FAE_20D ≈ **-2.89%**

信用市场快速恶化与未来更严重的股票 downside 在 DEV 中具有明显关系。

### VIX

`VIXLevel` 是 DEV 中最干净的单调关系之一。

从最低 VIX quintile 到最高 quintile，Mean FAE_20D 大致恶化为：

**-1.52% → -1.89% → -2.39% → -3.10% → -4.25%**

但高 VIX 同样可能属于风险状态确认，而不是领先预警。

------------------------------------------------------------------------

## 7. Equal-Weight Weakness：第一次明显的 OOS 失败

DEV 中，`EqualWeightRelative20` 最差的 Q1 对应明显更差的未来 FAE。

于是冻结 DEV bottom-20% threshold：

**EqualWeightRelative20 \< 约 -0.81%**

然后把这个固定阈值直接应用到 CONFIRM。

结果出现明显 distribution shift：

原本代表 DEV bottom 20% 的阈值，在 2023--2025 居然约有 **49.5%** 的
observations 满足。

更重要的是，预测方向反转。

CONFIRM：

-   Not Weak Mean FAE_20D：**-2.22%**
-   Weak Mean FAE_20D：**-1.65%**

因此 DEV 中看起来很有吸引力的 equal-weight weakness signal
没有样本外泛化。

研究没有重新 qcut CONFIRM 来重新制造一个 bottom 20%
signal，因为那等于使用 test data 重新训练规则。

------------------------------------------------------------------------

## 8. DEV 开发的 Credit Early Warning Rule

在 DEV 单变量结果之后，进一步提出一个更贴近实际投资的 secondary rule。

这个规则**不是原始 preregistered hypothesis**，而是明确标记为：

**DEV-developed candidate rule。**

冻结条件：

-   `HYOASChange20BP > +34 bp`
-   `SPYDistanceHigh60 >= -5.64%`
-   `VIXLevel <= 24.52`

经济含义是：

> 信用市场已经快速恶化，但 SPY 还没有出现严重 price damage，同时 VIX
> 也还没有进入明显 stress。

也就是尝试寻找真正的：

**Credit deteriorates first, equity/VIX have not fully reacted yet。**

### DEV 结果

Early Warning：

-   311 observations
-   Frequency：6.97%
-   Mean FAE_20D：**-3.26%**
-   Median FAE_20D：**-2.69%**

Normal：

-   Mean FAE_20D：**-2.58%**
-   Median FAE_20D：**-1.45%**

DEV 中这个 candidate rule 表现非常有吸引力。

------------------------------------------------------------------------

## 9. CONFIRM：Credit Early Warning 失败

完全不修改阈值，直接应用到 2023--2025。

### Normal

-   N = 715
-   Mean FAE_20D：**-1.97%**
-   Median：**-1.30%**

### Early Warning

-   N = 31
-   Mean FAE_20D：**-1.28%**
-   Median：**-0.62%**

关系不仅没有重复，而且方向完全反转。

因此：

> **DEV-developed Credit Early Warning 没有 OOS generalize。**

这不代表 Early Warning 实际具有"保护作用"；CONFIRM signal
数量太少，不能提出这种新的反向结论。

我们只能得出：

**现有 OOS evidence 不支持原来的 Early Warning hypothesis。**

------------------------------------------------------------------------

## 10. Overlap Robustness

`FAE_20D` 存在一个天然问题：

相邻两天的 label 最多共享 19/20 个未来交易日。

因此 daily observations 不能被简单理解成完全独立的实验。

为此进行了 20-offset approximately non-overlapping robustness check。

### DEV

-   Mean relationship：**18 / 20 offsets** 支持
-   Median relationship：**17 / 20 offsets** 支持
-   Average Mean Difference：**-0.68 percentage points**
-   Average Median Difference：**-1.15 percentage points**

### CONFIRM

-   Mean relationship：仅 **5 / 20**
-   Median relationship：仅 **5 / 20**
-   Average Mean Difference：**+0.82 percentage points**
-   Average Median Difference：**+0.25 percentage points**

因此可以排除一个重要解释：

> DEV 的漂亮结果并不只是 overlapping FAE labels 人为制造出来的。

更合理的结论是：

> DEV 中确实存在这种关系，但它没有跨时期稳定泛化到 2023--2025。

CONFIRM 每个 offset 的 warning observations
很少，因此具体数值需要谨慎解释。

------------------------------------------------------------------------

## 11. Independent Episode Analysis

另一个问题是：

> DEV 的 311 个 warning days 会不会其实只是几个长期 stress episode
> 被重复计算很多次？

因此把连续 True observations 合并为一个 episode。

### DEV

311 warning days → **75 个 episodes**

-   Median duration：**2 天**
-   Mean duration：**4.15 天**
-   第一次报警后的 Mean FAE_20D：**-2.25%**
-   Median：**-1.58%**
-   Worst：**-15.61%**

### CONFIRM

31 warning days → **8 个 episodes**

-   Median duration：**2 天**
-   Mean duration：**3.88 天**
-   第一次报警后的 Mean FAE_20D：**-0.69%**
-   Median：**-0.44%**
-   Worst：**-5.49%**

因此 DEV effect 在把重复 daily warnings 压缩成 independent episodes
后仍然存在。

但到了 CONFIRM，仍然没有复现。

CONFIRM 只有 8 个
episodes，所以估计精度有限；但现有样本外证据仍然没有支持该 signal。

------------------------------------------------------------------------

## 12. 最终结论

### Primary Hypothesis：NOT SUPPORTED

本研究没有找到稳定证据证明：

> **高 Treasury-rate pressure + weakening market resilience
> 能够可靠预测未来 20 个交易日更严重的 SPY downside。**

DEV 中确实出现了多个经济逻辑合理、统计表现也很漂亮的关系，包括：

-   SPY price damage
-   HY credit deterioration
-   elevated VIX
-   equal-weight weakness
-   Credit Early Warning

但最有希望的 candidate early-warning signals 没有通过 2023--2025
CONFIRM。

而且这个失败在以下 robustness checks 后仍然成立：

-   冻结 DEV thresholds；
-   不修改 20D outcome horizon；
-   20-offset non-overlapping check；
-   independent episode analysis。

因此最终接受：

**原始假设没有得到样本外支持。**

------------------------------------------------------------------------

## 13. 对实际投资的意义

POC03-A 不支持机械地因为以下条件出现就直接减仓：

-   10Y Treasury yield 很高；
-   RSP 严重跑输 SPY；
-   HY OAS 在 20 期扩大超过 34bp；
-   或本研究设计的 Credit Early Warning rule 亮灯。

这些指标仍然可以作为风险 dashboard 的输入。

但目前证据不支持：

**Signal = True → Mechanical Sell / De-risk**

避免采用一个看起来聪明、但 OOS
不稳定的风险规则，本身也是风险管理的一部分。

------------------------------------------------------------------------

## 14. 真正的研究发现：Non-stationarity / Regime Dependence

POC03-A 最重要的发现不是一个可交易 signal。

而是：

> **2005--2022 中非常强、经济逻辑也很合理的 cross-asset
> relationship，可以在 2023--2025 完全失效。**

因此问题从：

**Signal → Risk**

进一步变成：

**Regime → (Signal → Risk)**

也就是：

> 在什么市场 regime 下，Rates、Credit、VIX、Breadth / Market Internals
> 等 signal 才真正具有未来风险信息？

这是一个新的研究问题。

它不属于 POC03-A，也不会被用来事后挽救原来的假设。

如果继续研究，应作为独立项目，例如：

**POC03-B --- Regime Instability of Cross-Asset Risk Signals**

------------------------------------------------------------------------

## 15. Research Integrity

这篇研究保留失败结果。

CONFIRM 失败以后，没有：

-   修改 20D horizon；
-   改 +34bp / -5.64% / 24.52 的 frozen thresholds；
-   在 CONFIRM 重新 qcut；
-   删除不喜欢的年份；
-   换一个 label 直到结果成功；
-   加 Random Forest 等复杂模型来"救"原始假设。

研究路径是：

**Economic intuition → Frozen features → Frozen label → DEV → Candidate
rule → Frozen thresholds → CONFIRM → Failure → Robustness → Accept
failure**

最终的 failed OOS test 本身就是研究结果。

------------------------------------------------------------------------

## 16. 最值得保留的 ML4T 教训

**In-sample 漂亮不是成果，经济逻辑合理也不是成果。**

一个 relationship
只有在没有参与设计它的数据上仍然站得住，才开始有资格被称为 signal。

POC03-A
最终没有产生一个可以机械交易的风险规则，但它成功识别了一个更深层的问题：

**金融市场中的 feature → risk relationship 具有明显的非平稳性和 regime
dependence。**

这也是下一阶段研究最值得追踪的问题。

------------------------------------------------------------------------

**POC03-A STATUS: CLOSED**
