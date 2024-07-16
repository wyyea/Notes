1. 为何是双向RTT而不是单向的传播时延
2. 不是说在带宽受限阶段无法精准测量RTT吗，为什么无论什么情况都更新RTT
3. 如果探测到瓶颈带宽增大，pacing gain会保持>1，直到达到瓶颈带宽？而不是pacing gain一直简单循环？降低带宽也是保持<1？

![[Pasted image 20240713174301.png]]
为何低速情况下会卡住？

# v2:

1. 初始时$pacing\_gain = 2.77\neq 2 = cwnd\_gain$ ?
2. 什么是快速恢复，不是不使用原有的拥塞控制算法吗
![[Pasted image 20240714153545.png]]
3. 如何理解 long-term model 和 short-term model
4. 下文最后几句什么意思
![[Pasted image 20240714160238.png]]
5. 下文最后一句什么意思
![[Pasted image 20240715161626.png]]