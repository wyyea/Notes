# Modeling BBR’s Interactions with Loss-Based Congestion Control (BBR)
## 1 概要
 在 BBR 和其他 Loss-based 拥塞控制算法（CCA）的竞争中，无论 BBR 的数量和 Loss-based 流的数量对比是多少，BBR 总是占据接近一半的带宽 -> <mark style="background: #FF5582A6;">unfair</mark>
> ([[Modeling_BBR’s_Interactions_with_Loss-Based_Congestion_Control.pdf#page=2&selection=149,16,150,69|📖]])
>  BBR consumes an outsized share of bandwidth, leaving just over half to be shared by the sixteen other connections.

文章使用模型对 BBR 与 Loss-based 流竞争进行拟合
## 2 方法
1. 在 1 条 BBR 流和一条非 Loss-based 流竞争时，BBR 流在 ProbeBW 时会抢占带宽导致非 Loss-based 流丢包并减小窗口，从而 BBR 流得到更多带宽直到达到 inflight-cap
> ([[Modeling_BBR’s_Interactions_with_Loss-Based_Congestion_Control.pdf#page=3&selection=257,35,265,21|📖]])
> Loss-based algorithms back off, dropping their window sizes and corresponding sending rate. BBR does not react to losses and instead increases its sending rate, since it successfully sent more data during bandwidth probing than it did in prior cycles. 

2. 在 多条 BBR 流相互竞争过程中，带宽估算方法使其高估带宽，而 BDP 较小时，在 ProbeRTT 时未能排空队列从而高估 RTT
> ([[Modeling_BBR’s_Interactions_with_Loss-Based_Congestion_Control.pdf#page=4&selection=331,18,342,28|📖]])
> However, if 4 N packets is greater than the BDP, the queue will not drain during ProbeRTT so RTTest includes some queueing delay

多条 BBR 流和若干条 Loss-based 流竞争时，高估 RTT 导致最终得到 Loss-based 流获得不超过一半的带宽

3. 考虑 BBR 流 ProbeRTT 时段占用带宽较少，综合后得到模型与实验结果相近
## 3 结论
BBR 在 ProbeBW 时使用 inflight-cap 进行限制最大带宽，但该量只与 BDP 相关，未考虑其他流的数量，导致影响公平性

# When to use and when not to use BBR: An empirical analysis and evaluation study (BBR)
## 概要
对 BBR 和 Cubic 在不同 Bw，RTT，buffer size 下进行测量比较
## 方法
1. BBR 与 Cubic 的对比
+ BBR 在浅 buffer 下吞吐高于 Cubic, 因为浅 buffer 下两者都会丢包，而 BBR 对丢包反应不敏感, Cubic 在丢包后会减少窗口
+ 在浅 buffer 下 BBR 丢包会显著高于 Cubic, 这是因为 BBR 维持 2 x BDP 的 inflight, 只有 buffer >= BDP 时才能不丢包。而在大 buffer 下两者丢包都显著下降
+ 在浅 buffer 下 BBR 具有较低延迟。在深 buffer 且小 BDP 下 Cubic 延迟低于 BBR, 其余情况下 BBR 一般有较低的延迟

2. BBR 的丢包率分界点
	在 BBR 丢包率超过分界点 20% 后, BBR 的吞吐会大幅下降，这是因为带宽探测只能越探越小
> ([[When_to_use_and_when_not_to_use_BBR:_An_empirical_analysisEand_evaluation_study.pdf#page=5&selection=89,2,114,1|📖]])
> Thus, if this value is less than the bandwidth, BBR will not probe for additional capacity, and will in fact infer a lower capacity due to losses. We determine the cliff point by solving: 
> 				pacing_gain × BW × (1 − p) = BW 
> 				p = 1 - 1 / pacing_gain

　　但分界点无法超过 20%, 这是因为 BBR 在丢包率达到 20% 后不再使用探测带宽进行更新，而是使用 long-term average BW 代替
  
３. BBR 与 Cubic 共存时对带宽的使用
+ 浅 buffer 下 BBR 使用绝大部分带宽，因为其重传量显著高于 Cubic
+ 深 buffer 时 Cubic 能占据不小于或超过 BBR 的带宽，因为 BBR 的重传大大减少
## 局限性
+ 只使用了简单拓扑
+ 只是经验测量，未改进 BBR 机制使解决问题