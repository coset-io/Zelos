# v2_20250804.pdf

本文会回答澄清非常多的V2问题，例如：

1. LVR无常损失的定义与问题
2. 永续期权和uniswapV2提供流动性的关系
3. **提供流动性和波动率没有关系**

## 前置知识

我们假设你1.对uniswapV2 有了一些基础了解2.有一些随机过程尤其是伊藤引理的知识3.Shreve的《随机金融分析》中对永续美式put 定价的知识也会使用到。

## V2的基本问题陈述

### 资产价格

我们假定交易为ETH/USDC,价格服从几何布朗运动。在风险中性测量下，$S_t$遵循几何布朗运动：

$$
dS_t = rS_tdt + \sigma S_tdW_t
$$

其中：

- $r$是无风险利率。
- $\sigma$是价格波动率。
- $W_t$是标准布朗运动。

初始价格$S_0$已知。

### 定义LP价值过程

和常见的V2数据相比，我们设定我们的初始投资金额为$1。我们稍加改造可以得到

$$
LP(S_0) = 1
$$

$$
LP(S_t) = \sqrt{S_t}
$$

此外我们还假设我们会因为提供流动性获得手续费收入C。

### 随机过程推导

直接进行高效的随机过程分析。我们先假设投资组合$V_t$不包含手续费。

$$
V_t = LP(S_t)
$$

使用伊藤引理，推导$V_t$的SDE：

- $V_t = S_t^{1/2}$
- 计算偏导数：

$$
\frac{\partial V}{\partial S} = \frac{1}{2} S^{-1/2},
$$

$$
\frac{\partial^2 V}{\partial S^2} = -\frac{1}{4} S^{-3/2}
$$

- 应用伊藤引理：

$$
dV_t = \frac{\partial V}{\partial S} dS_t + \frac{1}{2} \frac{\partial^2 V}{\partial S^2} (dS_t)^2
$$

代入$dS_t = rS_tdt + \sigma S_tdW_t$和$(dS_t)^2 = \sigma^2 S_t^2 dt$。

简化：

$$
dV_t = \left[ \frac{1}{2} S^{-1/2}(rS_tdt + \sigma S_tdW_t) \right] + \frac{1}{2} \left[ -\frac{1}{4} S^{-3/2} \right] \sigma^2 S_t^2 dt
$$

$$
dV_t = \frac{1}{2} S^{1/2}(rdt + \sigma dW_t) - \frac{1}{8} S^{1/2} \sigma^2 dt
$$

代入$V_t = S^{1/2}$，最终：

$$
dV_t = \frac{\sigma}{2} V_tdW_t + \left( \frac{r}{2} - \frac{\sigma^2}{8} \right) V_tdt
$$

非常好，非常好。在这里我们遇到了第一个老朋友——LVR（Loss Versus Rebalancing，无常损失的新定义）。论文中给出的V2相关LVR公式如下：

---

**Example 3 (Constant Product Market Maker / Uniswap v2).** Taking $\theta = 1/2$ in Example 2, we have

```
[^evans2000]: in the Black-Scholes framework, the delta of an option depends on volatility and other parameters. See also Proposition 1 of Evans [2000], evaluating a weighted geometric mean market maker over a finite horizon using risk-neutral pricing.
```

that

$$

V(P) = 2L\sqrt{P}, \quad \ell(\sigma, P) = \frac{L\sigma^2}{4} \sqrt{P}, \quad \frac{\ell(\sigma, P)}{V(P)} = \frac{\sigma^2}{8}.
$$

(16)

如果参考 Anthony Lee Zhang 的论文https://anthonyleezhang.github.io/pdfs/lvr.pdf

可以发现作者关注的是 LP 的瞬时损失。LVR本质上衡量的是机会成本，即"如果我将资金用于最优对冲策略会怎样"。因此，论文中通常设定 $r = 0$，只关注波动带来的损失。在我们的推导中，dt 项对应的正是 $\frac{\sigma^2}{8}$，这与 LVR 公式中的瞬时损失一致。读到这里值得停下来庆祝一下，我们通过随机分析的捷径更快地推导到了 LVR。下面我们要讨论无常损失了。

## 无常损失

无常损失在抱怨什么呢？"如果是当时买下 ETH 了该多好。"这对随机过程来说也不难。

接下来我们可以直接将 LP 价值过程的 SDE 与单边持有资产 $S_t$ 进行比较：

- 对于 Buy & Hold 策略，资产价值过程为：
  
  $$
  S_t = S_0 e^{rt + \sigma W_t}
  $$
  
  若 $S_0 = 1$，则：
  
  $$
  S_t = e^{rt + \sigma W_t}
  $$

其期望值为：

$$
\mathbb{E}[S_t] = e^{rt}
$$

- 对于 LP 策略，价值过程为：
  $$
  V_t = \exp\left[\left(\frac{r}{2} - \frac{\sigma^2}{8}\right)t + \frac{\sigma}{2}W_t\right]
  $$

其期望值为：

$$
\mathbb{E} [V_t]= \exp\left\{\frac{r}{2}t\right\}
$$

可以看到，LP 的期望收益率仅为 $r/2$，远低于 Buy & Hold 的 $r$。这正是无常损失的本质：LP 的长期收益率低于单边持有。

进一步考虑手续费的影响。假设手续费以常数速率c线性累计，则 LP 策略的长期平均年化收益率为：

$$
\frac{r}{2} + c
$$

回顾一下 C 的定义是 价值 1 udsc 的流动性提供对应的手续费回报。数值上也可以理解为流动性提供的回报率。

与 Buy & Hold 策略的年化收益率进行比较，得到边界条件：

$$
\frac{r}{2} + c > r
$$

即：

$$
c > \frac{r}{2}
$$

也就是说，只有当单位时间手续费率$c$大于无风险利率的一半时，LP 策略的长期平均收益率才能超过 Buy & Hold。

很好，我们已经掌握了随机过程版本的 IL 和 LVR 了。但是其实还是有一些问题没有解决。看上去这是一个衍生品，但是又没有BSM 那样的公式指导定价和 greek 分析。接下来我们将进行最革命性的推导和反直觉的结论。一切都要从永续美式期权定价开始讨论。

## 初步停时定价公式

通过观察

$$
\mathbb{E} [V_t]= \exp\left\{\frac{r}{2}t\right\}
$$

你可能会以为这就是我们对 Uniswap V2 头寸价值的定价，但其实它并不是一个真正的价格公式。为什么呢？首先，它只描述了未来价值的平均增长趋势，却忽略了贴现效应——也就是说，它没有考虑“未来的一块钱在今天值多少钱”。更重要的是，它没有涉及决策：我们应该什么时候退出？是立刻赎回还是等待更久？真正的定价必须结合策略，就像期权定价中，我们关心的不是平均收益，而是如何在最佳时机行权、最大化回报。这个公式只是一个“趋势预估”，并不能告诉我们该怎么做，也无法定义当前的合理价值。

我们需要通过有决策参与的角度考虑V2头寸定价，从而得到一个完备有效的定价公式。

在 Uniswap V2 中，流动性头寸没有固定的到期日，你可以一直持有它。这就像你拥有一个“没有期限的资产”，只要你愿意，什么时候退出都可以。所以，不同于传统期权，期权越接近到期日，时间价值就越少。V2头寸在两个不同时间点 $t$ 和 $t'$下，可以认为都距离到期日有无穷多的时间，它们在时间价值上是相同的。因此，市场条件一样，即 $S_t = S_{t'}$条件下，这两个时间点的头寸从投资角度来看是一样的——没有一个比另一个更“接近到期”或“更有价值”。 V2 头寸没有这种“时间压力”，所以只看当前市场状态（比如价格），而不是挂念“还剩几天到期”。这就是为什么我们说，它没有“期限感”，更关注的是当前，而不是未来。

因此，我们围绕以$S_t$为状态的动态规划策略，类比美式永续期权定价框架来进行接下来的工作。

我们设定预设一个价格边界 $L$，当当前价格$S_t$越过边界时立即行权。发现我们不能预知未来价格，只能根据当前价格决定是否行权，同时当前价格已经包含过去的所有信息，这完全符合一个停时(stopping time) 的定义。

遍历所有策略即所有停时的可能，得到V2头寸价值公式

$$
V^*(x)= \sup_{\tau \in \mathcal{T}}\left\{\mathbb{E}\left[e^{-r\tau}\cdot\text{payoff}(\tau)\right]\right\} , \quad x=S_0
$$

若永不行权则价值为0；若立即行权则

$V(S_0) = \text{payoff}(S_0) = \sqrt{S_0}$

接下来，我们只需要找到最优停时，即找到一个最优边界 $L^*$ 即可得到一个完备的价格公式。

考虑

$$
V_L(x) = \mathbb{E}\left[e^{-r\tau_L}\cdot\sqrt{S_{\tau_L}} \right]
$$

显然对于$L\leq S_0$，我们的收益总是不如单纯持有ETH，故设置 $L>S_0$，也就是一个向上停时

$$
V_L(S_0) = \mathbb{E}\left[e^{-r\tau_L}\cdot\sqrt{S_{\tau_L}} \right]
$$

$$
=\sqrt{L}\cdot \mathbb{E}\left[e^{-r\tau_L}\right]
$$

$$
=\sqrt{L}\cdot\frac{S_0}{L}
$$

$$
=\frac{S_0}{\sqrt{L}}
$$

其中$\mathbb{E}\left[e^{-r\tau_L}\right]=\frac{S_0}{L}$的计算过程见附录1

我们发现这是一个随$L$增大而减小的函数，故最佳决策是立即行权，单纯的V2头寸没有等待的价值。

回忆 $\mathbb{E} (V_t)=\exp\left\{\frac{r}{2}t\right\}$，说明LP收益不如单纯持有ETH的特点是与策略无关的。

因此我们需要改变payoff即引入手续费补偿LP来改善这一点

## 持续引入现金流模型

V2内的每笔交易都会向交易者收取一定比例的手续费，手续费以留在池子内增加总储备量的形式来补偿LP。这笔伴随$S_t$价格变化产生的手续费等价于每时每刻LP都会收到一小笔现金流，速率为$C$。

我们重新建立持续收入现金流$C$的模型

$$
V(S_0) = \sup_{\tau \in \mathcal{T}} \left\{ \mathbb{E} \left[ e^{-r\tau}\cdot\sqrt{S_\tau} + \int_0^\tau e^{-rt}\cdot C\,dt \right] \right\}
$$

其中 $\int_0^\tau e^{-rt}\cdot C\,dt=\frac{C}{r}(1-e^{-r\tau})$为补偿的贴现。

此时若立即行权则价值为$\sqrt{S_0}$；若永不行权则价值为 $\frac{C}{r}$。

对于 $\Delta t \to 0$

Case1: 行权 得到收益 $V(S)=\sqrt{S}$
Case2: 不行权 $V(S) \approx C\Delta t + e^{-r\Delta t} \cdot\mathbb{E} V(S_{\Delta t})$

对于$V(S_t)$ 由伊藤引理我们有

$$
dV= V'(S_t)dS_t +\frac{1}{2}V''(S_t)(dS_t)^2
$$

$$
= \frac{\partial V}{\partial S}(rS_tdt+\sigma S_tdW_t)+ \frac{1}{2}\frac{\partial^2 V}{\partial S^2}\sigma^2 S_t^2 dt
$$

$$
=\left(rS_tV'(S_t)+\frac{1}{2}\sigma^2 S_t^2 V''(S_t)\right)dt+\sigma S_t V'(S_t)dW_t
$$

考虑到无套利情况：$rV(S) = C+rSV'(S)+\frac{1}{2}\sigma^2 S^2 V''(S)$
我们得到HJB方程

$$
0 = \max\left\{\sqrt{S}-V(S), \frac{1}{2}\sigma^2 S^2 V''(S)+rSV'(S)-rV(S)+C\right\}
$$

因此我们可以得到ODE

$$
\frac{1}{2}\sigma^2 S^2 V'' + rSV' - rV + C = 0
$$

解得

$$
V(S) = \begin{cases} 
\sqrt{S} & \text{当 } S \geq \left(\frac{2C}{r}\right)^2 \\
\frac{r}{4C}S + \frac{C}{r} & \text{当 } S < \left(\frac{2C}{r}\right)^2 
\end{cases}
$$

(具体证明过程见附录2)

使用python进行蒙特卡洛模拟（详细代码见附录3）
我们得到差异值小于 $ 10^{-3} $ ，数值上验证公式正确。

观察公式，我们得到一个非常反直觉的结论，即V2头寸价值与波动率无关。

下面我们来解释为什么会有这种情况发生
根据Jensen不等式，对于凸函数$f(x)$我们有

$$
f(\mathbb{E}[x]) \leq \mathbb{E}[f(x)]
$$

这意味着对于凸的payoff 函数$f(S_t)$ ，如  传统期权行权时的payoff ：$f(S_t)=\max(S_t-K,0)$），其预期值的"中点"直接代入函数所得的值更高。因此引入不确定性时$\mathbb{E}[f(x)]$ 会增大。

而对于凹函数其Jensen不等式为

$$
f(\mathbb{E}[x]) \geq \mathbb{E}[f(x)]
$$

这意味着对于凹的payoff函数$f(S_t)$，也就是V2中的$\sqrt{S_t}$。$\mathbb{E}[f(S_t)]$ 不会收益于波动率的升高，反而可能降低。而对于波动率惩罚的部分，由$S_t$决定的动态规划带来了最优行权边界$S^*$，也就是得到一个最优停时$\tau^*$。我们可以选择是否行权，何时（$S_t$为多少时）行权，这种"择时权"可以对冲原始payoff的凹性带来的波动率惩罚。

而对于引入现金流的模型，payoff仍然是一个关于$S_t$的凹函数，并与波动率无关。$C$的参与只使得最优策略发生变化，不再是原先的立即行权，而是存在一个新的最优行权边界。

从公式的角度讲，我们在解ODE时因为会冲突边界条件而去掉了含有波动率的特征根，使得最终表达式不含波动率。

值得一提的是，虽然我们发现 V2 头寸的定价结果与波动率无关，但这其实暴露了模型的一个小缺陷：我们没有考虑建仓成本。

现实中，构建流动性头寸是需要手续费或其他成本的，尤其当波动率很高、价格剧烈波动时，LP 可能频繁达到“退出条件”并重新建仓，这些反复操作的成本会随着波动率上升而增加。比如说，每次你往池子里添加流动性，都要支付一次建仓成本（比如手续费），假设是 1 块钱。如果价格波动很小，你可能很长时间都不需要重新建仓，成本也就摊薄了。但如果波动率很高，价格经常触发退出条件，你就需要频繁重新建仓，每次都要再付那 1 块钱。这样一来，随着波动率增加，你实际花出去的总成本也会增加。但我们的模型默认你“免费”无限次建仓，这显然低估了波动率带来的实际支出。也正因如此，我们才会在数学上看到一个“与波动率无关”的结果——因为建仓这块成本完全被忽略了。这是一个值得在未来模型中修正的地方。

## 总结

|  | 无常损失 | LVR | 最优停时定价 |
| --- | --- | --- | --- |
|机会成本  |与买入并持有同一资产组合相比  | 与可随时套利、自动再平衡的理想交易者相比 | 无风险利率，风险中性定价
|  界定条件| 价格上下极限决定最大损失，路径无关 |路径相关，强调动态套利机会  | 边界由最优行权边界决定|
|适用场景  | 说明AMM与简单持有的性能差异 | 用于分析LP被套利者"占便宜"的程度 |定价V2头寸，指导真实决策 |

## 附录

1. 

**引理 .** 假设 $r > 0$，则有：

$$
\mathbb{E}^* \left[ e^{-r\tau_L} \right] = \frac{S_0}{L}.
$$

**证明.** 只需考虑 $S_0 = x < L$ 的情况。注意到对于所有 $\lambda \in \mathbb{R}$，过程 $(Z^{(N)}_t)_{t \in \mathbb{R}_+}$ 定义为：

$$
Z^{(N)}_t := (S_t)^\lambda e^{-r\lambda t + \lambda \sigma^2 t/2 - \lambda^2 \sigma^2 t/2} = (S_0)^\lambda e^{\lambda \sigma W_t - \lambda^2 \sigma^2 t/2}, \quad t \geq 0,
$$

在风险中性概率测度 $\mathbb{P}^*$ 下是一个鞅。因此，停时过程 $(Z^{(N)}_{t \wedge \tau_L})_{t \in \mathbb{R}_+}$ 也是一个鞅，并且具有恒定期望，即：

$$
\mathbb{E}^* \left[ Z^{(N)}_{t \wedge \tau_L} \right] = \mathbb{E}^* \left[ Z^{(N)}_0 \right] = (S_0)^\lambda, \quad t \geq 0. \quad (15.21)
$$

选择 $\lambda$ 使得：

$$
r = r\lambda - \lambda \frac{\sigma^2}{2} + \lambda^2 \frac{\sigma^2}{2},
$$

即：

$$
0 = \lambda^2 \frac{\sigma^2}{2} + \lambda \left( r - \frac{\sigma^2}{2} \right) - r = \frac{\sigma^2}{2} \left( \lambda + \frac{2r}{\sigma^2} \right) (\lambda - 1),
$$

关系式 (15.21) 可重写为：

$$
\mathbb{E}^* \left[ (S_{t \wedge \tau_L})^\lambda e^{-r(t \wedge \tau_L)} \right] = (S_0)^\lambda, \quad t \geq 0. \quad (15.22)
$$

选择正解 $\lambda_+ = 1$，得到以下界限：

$$
0 \leq Z^{(N_+)}_{t} = e^{-rt} S_t \leq S_t \leq L, \quad 0 \leq t \leq \tau_L, \quad (15.23)
$$

因为 $r > 0$ 且 $S_t \leq L$ 对所有 $t \in [0, \tau_L]$ 成立。因此有 $\lim_{t \to \infty} Z^{(N_+)}_{t \wedge \tau_L} = Z^{(N_+)}_{\tau_L}$ 在 $\{\tau_L < \infty\}$ 上。由 (15.22)-(15.23) 和控制收敛定理，可得：

$$
L\mathbb{E}^* \left[ e^{-r\tau_L} \right] = \mathbb{E}^* \left[ e^{-r\tau_L} S_{\tau_L} I_{\{\tau_L < \infty\}} \right] = \mathbb{E}^* \left[ \lim_{t \to \infty} e^{-r(t \wedge \tau_L)} S_{t \wedge \tau_L} \right] = \mathbb{E}^* \left[ \lim_{t \to \infty} Z^{(N_+)}_{t \wedge \tau_L} \right] = \lim_{t \to \infty} \mathbb{E}^* \left[ Z^{(N_+)}_{t \wedge \tau_L} \right] = \lim_{t \to \infty} (S_0)^\lambda = S_0,
$$

从而得到：

$$
\mathbb{E}^* \left[ e^{-r\tau_L} \right] = \frac{S_0}{L}. \quad (15.24)
$$

2. 

通解 $V_h = A S^{\lambda_1} + B S^{\lambda_2}$

特解 $V_p= \frac{C}{r}$

所以$V(S) = A S^{\lambda_1} + B S^{\lambda_2}+\frac{C}{r}$

讨论通解，可以列出特征方程

$$
\sigma^2\lambda^2 + (2r-\sigma^2)\lambda -2r =0
$$

代入 $\lambda_1=1$，发现是其中一个解
由韦达定理 $\lambda_2 = (\lambda_1+\lambda_2)-\lambda_1 = -\frac{2r-\sigma^2}{\sigma^2}-1=-\frac{2r}{\sigma^2}<0$

讨论边界条件当$S\to 0^+$时，我们倾向于不行权，$V(S)=\frac{C}{r}$
为了能达到此边界条件我们需要令$B=0$
因此我们有：

$$
V(S) = \begin{cases}
A S + \frac{C}{r} & \text{当 } S < S^* \\
\sqrt{S} & \text{当 } S \geq S^*
\end{cases}
$$

从数学角度来看，解ODE要求函数和其一阶导是连续的。
从金融角度来看， 如果一阶导数边界连接不平滑，
1当$V'(S^+)>V'(S^-)$时，等待对于$V(S)$增长没有什么帮助，LP不如早点行权拿到payoff

2当$V'(S^+)<V'(S^-)$时，继续持有价值增长比立即行权要快，LP会后悔自己太早行权

因此只有$V'(S^+)=V'(S^-)$也就是一阶导连续时，才符合最优行权边界的定义，没有套利空间或后悔行为。

利用smooth pasting这一特点，我们解得

$$
A=\frac{r}{4C}, \quad S^* = \left(\frac{2C}{r}\right)^2
$$

即

$$
V(S) = \begin{cases} 
\sqrt{S} & \text{当 } S \geq \left(\frac{2C}{r}\right)^2 \\
\frac{r}{4C}S + \frac{C}{r} & \text{当 } S < \left(\frac{2C}{r}\right)^2 
\end{cases}
$$

3. 

```python
import numpy as np

# 参数设置
S0 = 1.0
r = 0.04
sigma = 0.2
C = 0.01
T = 100 # 总模拟时间（足够大）
dt = 0.01 # 时间步长
N = int(T / dt)
M = 100000 # 模拟路径数

# 最优行权边界 S*
S_star = (2 * C / r) ** 2

# Analytical formula
def V_analytical(S):
    if S >= S_star:
        return np.sqrt(S)
    else:
        return 0.5 * (r / (2 * C)) * S + C / r

# 模拟 S(t) 路径
np.random.seed(42)
S = np.full(M, S0)
discounted_cashflows = np.zeros(M)
stopped = np.zeros(M, dtype=bool)
running_discount = np.zeros(M)
time = 0.0

for i in range(N):
    if np.all(stopped):
        break
    Z = np.random.normal(size=M)
    S = S * np.exp((r - 0.5 * sigma ** 2) * dt + sigma * np.sqrt(dt) * Z)
    time += dt

    # 计算每条路径是否达到停止条件
    trigger = (S >= S_star) & (~stopped)
    # 对未停止路径累加贴现手续费
    running_discount[~stopped] += C * np.exp(-r * time) * dt
    # 对达到停止条件路径，计算 sqrt(S) * e^{-r tau}
    discounted_cashflows[trigger] = np.sqrt(S[trigger]) * np.exp(-r * time) + running_discount[trigger]
    stopped[trigger] = True

# 对于从未触发行权的路径（极少数），设为只收手续费
discounted_cashflows[~stopped] = running_discount[~stopped]

# Monte Carlo 估值
V_mc = np.mean(discounted_cashflows)
V_theory = V_analytical(S0)

print(f"Monte Carlo Value: {V_mc:.6f}")
print(f"Theoretical Value: {V_theory:.6f}")
print(f"Difference: {abs(V_mc - V_theory):.6f}")
```


