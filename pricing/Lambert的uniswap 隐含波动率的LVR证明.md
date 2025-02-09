
## uniswap v3隐含波动率 
和期权的隐含波动率类似，我们需要假设流动性提供者没有赚到超额利润，进而推理出其金融活动对应的波动率观点。

如果模型能刻画流动性提供者的收入，并且其参数里有波动率，我们就可以尝试推导出该模型下的隐含波动率。


### Lambert iv

在2024年11月，panoptic research 发布了  From CEX to DEX: Comparing Uniswap and Deribit Implied Volatilities。https://panoptic.xyz/research/new-formulation-implied-volatility 中给出了详细的推导过程。
最终给出了其隐含波动率公式：

$$
\sigma_{panoptic} = 2* feeRate*\sqrt{\frac{Volume}{tickLiqudity}}
$$

### LVR

原版的论文（a16z）：https://arxiv.org/pdf/2208.06046
手续费速率（zelos）：https://medium.com/zelos-research/how-to-use-lvr-for-pool-selection-c442601c233f

论文里给出了瞬时LVR 的公式，我们需要进一步整理推导出LVR 版本的无常损失。我们只需要让收入“速率” = 损失“速率”即可。
绝大部分的难点在原版的论文给出的瞬时LVR。我们只需稍加加工：

论文里给出了uniswapv3 的瞬时LVR 为，K为头寸的流动性。

$$
l(\sigma,P) = \frac{K\sigma^2}{4}\sqrt{P}
$$

我们对其做了归一化处理 $L=\frac{𝑆_𝐿}{𝑆_0} 𝐻=\frac{𝑆_𝐻}{𝑆_0}$，以及让头寸的价值设定为 $1。

瞬时LVR则变为

$$
l(\sigma) = \frac{\lambda \sigma^2}{4}
$$

$$
\lambda = \frac{1}{2-\sqrt{L}-\frac{1}{H}}
$$

那么手续费获取速率部分也和$\lambda$ 相关。

$$
fee = C*\lambda
$$

其中C的意义是1单位 的流动性的手续费回报率。
最后我们让两者手续费获取速度 和 LVR 损失速度相等，得到：

$$
\sigma_{LVR} = 2\sqrt{C}
$$

$$
C = \frac {FeeRate \times 10 ^ {\frac{d0 + d1}{2}}}{\sqrt{P} \times Liq}
$$

其中：
* P是普通价格, 如1234.5678 usdc/eth
* FeeRate是手续费率
* d0和d1是token0和token1的decimal
* Liq是当前池的总流动性


### proof
#### LVR
目前我们有两个手续费的公式了。其实两者是一样的。
我们先从LVR 的iv 进行代换

$$
\sigma_{LVR} = \frac{2 \times \sqrt{FeeRate} \times 10^\frac{d0+d1}{2}}{\sqrt{Liq} \times p^{\frac{1}{4}}}
$$

#### panoptic
panoptic 这样我们能从代码中整理其数据得到（假设：token1是quote token）

$$
tickLiq = \frac {Liq \times FeeRate \times \sqrt{1.0001^{tick}}}{10^{d1}}
$$

如果遵照我们之前的假设, 也就是总价值为1u, 而且token1是quote token. 所以

$$
volume = \frac {1}{10^{d1}}
$$

合并可知:

$$
\sigma_{panoptic} = 2 \times FeeRate \times \sqrt{\frac{\frac{1}{10^{d1}}}{\frac {Liq \times FeeRate \times \sqrt{1.0001^{tick}}}{10^{d1}}}} \\
$$

$$
= 2 \times \sqrt{FeeRate} \times \frac{1}{\sqrt{Liq}} \times \frac{10^{\frac{d1}{2}}}{1.0001^\frac{tick}{4}}
$$

拼图的最后一块则是 $tick$ 和价格转换。

$$
P= 1.0001^{tick} \times 10^{d0-d1}
$$

$$
\frac{10^{\frac{d1}{2}}}{1.0001^\frac{tick}{4}} = \frac{10^{\frac{d1}{2}}} {\frac{P^\frac{1}{4}}{10^{\frac{d0-d1}{4}}}} = \frac{10^{\frac{d0+d1}{4}}} {P^\frac{1}{4}}
$$

带入回去得到

$$
\sigma = 2 \times \sqrt{FeeRate} \times \frac{1}{\sqrt{Liq}} \times \frac{10^{\frac{d0+d1}{4}}} {P^\frac{1}{4}} 
$$

$$
=\frac{2 \times \sqrt{FeeRate} \times 10^\frac{d0+d1}{2}}{\sqrt{Liq} \times p^{\frac{1}{4}}}
$$

所以，两者是完全一致的。


### 为什么是一样的

这不是从天而降的巧合，而是Lambert 和LVR使用的假设本质上是一致的，自然结论也应当一致。比如
1. set r =0
2. LVR 说的瞬时和 Lambert单点流动性实质上也是一回事。
### one more thing
我们不难发现，iv 公式最后只包含当前流动性和交易量两个变量。难道做市范围 (0.5,1.5) 的头寸和 （0.2,2.0) 对波动率的观点是一样的嘛？我们论文可能可以解决这个问题。https://arxiv.org/abs/2411.12375





