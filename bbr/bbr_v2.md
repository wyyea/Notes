---
id: bbr_v2
aliases: []
tags: []
---

# 算法细节
## 启动(Startup)
### 主要方法
+ 使用```BBRStartuppacintGain(2.77)```和```BBRStartupCwndGain(2)```, 经过$O(log_2(BDP))$轮找到```BBR.max_bw```
+ 探测到最大带宽后可能的队列深度不超过```(cwnd_gain - 1) * estimated_BDP = 1.77 * estimated_BDP```
+ 使用两个方面判断是否管道已满: ```BBR.max_bw```出现平坦区域，丢包

### ```BBR.max_bw```出现平坦区域
+ ```BBR.max_bw```曲线出现平坦区域，即不再发生增长
+ 连续三轮都增长都不超过25%，三轮可以确定平坦区域不是由接收窗口暂时导致的，三轮内接收窗口一般能够增大(```receive-window auto-tuning```)

### 丢包
+ 下列三个情况同时满足：
  + 连接已经进入**fast recovery**至少一个RTT
  + 丢包率超过```BBRLossThresh(2%)```至少一个RTT
  + 在```BBRStartupFullLossCnt=3```个非连续区域发生丢包
+ 三个情况同时满足能过滤掉突发丢包的因素

## 排空(Drain)
### 主要方法
+ 使用```pacing_gain = 1/BBRStartupCwndGain < 1```，期望一轮内排空队列
+ ```cwnd_gain = BBRStartupCwndGain```不变

### 退出条件
+ ```packets_in_flight <= BDP```
+ 退出后进入ProbeBW

## 探测带宽(ProbeBW)
### 主要方法
+ 轮流进行：以更高速率发送，以更低速率发送，以相同速率发送
+ 按序经过4个状态：DOWN，CRUISE，REFILL，UP

### ProbeBW_DOWN
+ 降低发送速率，设置```pacing_gain=0.9```
+ 退出条件：下列两项同时满足，退出后进入CRUISE
  + 存在**free headroom**，```packets_in_flight <= BBRHeadroom * BBR.inflight_hi```，留出空闲带宽用于带宽分享或突发容忍
  - [ ] (1-BBRHeadroom ?)      
  + 瓶颈链路的队列已排空，```packets_in_flight <= BBR.BDP```

### ProbeBW_CRUISE
+ 以相同速率发送，设置```pacing_gain = 1.0```
+ 会对丢包做出反应，降低```BBR.bw_lo, BBR.inflight_lo```，丢包指示了带宽和管道容量可能降低
+ 退出条件：到达需要进行带宽探测的时间，退出后进入REFILL

### ProbeBW_REFILL
+ 将```bw_lo, inflight_lo```设置为```infinity```，只通过```bw_hi, inflight_hi```来限制连接
+ 设置```pacing_gain = 1.0```，先将链路填满，避免在buffer较浅的情况下，刚进入UP阶段就收到丢包且此时链路未填满(丢包由```pacing_gain=1.25```引起)，从而使```inflight_hi```被低估
+ 退出条件：一个RTT后即退出(一般能够填满链路)，进入UP

### ProbeBW_UP
+ 提高发送速率，设置```pacing_gain = 1.25```，探测可用带宽的增加
+ ```bw_hi```：在丢包率不超过```BBRLossThresh```时，将```bw_hi```设置为新探测到的更大的值(若新值更大)
+ ```inflight_hi```：增长先慢后快，增长量按指数增长
  + 慢是因为可能余量十分紧凑
  + 快是因为可能可用带宽大大增加
+ 退出条件：下列任意条件满足
  + 已经保持在ProbeBW_UP至少1个```min_rtt```，且队列已足够大，即```packets_in_flight > 1.25 * BBR.bdp```
  + 丢包率超过```BBRLossThresh(2%)```

### ProbeBW的周期考虑
+ ProbeBW会影响其他基于丢包的拥塞控制算法(CUBIC/Reno)
+ 目标：
  + BBR能够正常探测实时带宽
  + 允许Reno/CUBIC达到BBR相同的带宽(在low BDP情况下)
+ ![time_scale_1.png](bbr/attachments/time_scale_1.png) ![time_scale_2.png](bbr/attachments/time_scale_2.png)
+ BBR-native
  + 超过2s, 使Reno能够正常增长到RTT=30ms,带宽=25Mbps(4K video)从BDP到2*BDP而不发生丢包
  + 小于3s，使BBR能够及时探测实时带宽
+ Reno-coexistence
  + 小于60轮，Reno/CUBIC难以达到4K video的带宽
  + 不超过65轮，从而使BBR能够容忍1%丢包率且能够在REFILL恢复到最大带宽(```BBW.max_bw * (1 - loss_rate)^N * 2 ~= 1```)
+ 上述策略使在low BDP时BBR与Reno/CUBIC相近，在large BDP时有较短的探测间隔
+ 控制从DOWN/CRUISE到REFILL的代码
```
/* Is it time to transition from DOWN or CRUISE to REFILL? */
  BBRCheckTimeToProbeBW():
    if (BBRHasElapsedInPhase(BBR.bw_probe_wait) ||
        BBRIsRenoCoexistenceProbeTime())
      BBRStartProbeBW_REFILL()
      return true
    return false
  /* Randomized decision about how long to wait until
   * probing for bandwidth, using round count and wall clock.
   */
  BBRPickProbeWait():
    /* Decide random round-trip bound for wait: */
    BBR.rounds_since_bw_probe =
      random_int_between(0, 1); /* 0 or 1 */
    /* Decide the random wall clock bound for wait: */
    BBR.bw_probe_wait =
      2sec + random_float_between(0.0, 1.0) /* 0..1 sec */
  BBRIsRenoCoexistenceProbeTime():
    reno_rounds = BBRTargetInflight()
    rounds = min(reno_rounds, 63)
    return BBR.rounds_since_bw_probe >= rounds
  /* How much data do we want in flight?
   * Our estimated BDP, unless congestion cut cwnd. */
  BBRTargetInflight()
    return min(BBR.bdp, cwnd)
```
    
### 代码
```
/* The core state machine logic for ProbeBW: */
  BBRUpdateProbeBWCyclePhase():
    if (!BBR.filled_pipe)
      return  /* only handling steady-state behavior here */
    BBRAdaptUpperBounds()
    if (!IsInAProbeBWState())
      return /* only handling ProbeBW states here: */
    switch (state)
    ProbeBW_DOWN:
      if (BBRCheckTimeToProbeBW())
        return /* already decided state transition */
      if (BBRCheckTimeToCruise())
        BBRStartProbeBW_CRUISE()
    ProbeBW_CRUISE:
      if (BBRCheckTimeToProbeBW())
        return /* already decided state transition */
    ProbeBW_REFILL:
      /* After one round of REFILL, start UP */
      if (BBR.round_start)
        BBR.bw_probe_samples = 1
        BBRStartProbeBW_UP()
    ProbeBW_UP:
      if (BBRHasElapsedInPhase(BBR.min_rtt) and
          inflight > BBRInflight(BBR.max_bw, 1.25))
        BBRStartProbeBW_DOWN()
```

## 探测往返时延(ProbeRTT)
### 主要方法
+ 在估计```BBR.min_rtt```之前，先进入```ProbeRTT```阶段，排空队列，避免队列增长导致RTT增长从而最大可容许cwnd增长，最终队列增长，形成循环
+ 设置```cwnd_gain = BBRProbeRTTCwndGain = 0.5```
+ 每过```MinRTTFilterLen=10s = 2 * ProbeRTTInterval```更新一次```BBR.min_rtt```，使用两段```ProbeRTT```的测量结果来更新，更具鲁棒性
+ 进入条件：未进行```ProbeRTT```超过```ProbeRTTInterval=5s```
+ 退出条件：维持```packets_in_flight = BBRProbeRTTCwndGain * BBR.bdp```至少```ProbeRTTDuration(200ms)```
  + 若```filled_pipe == true```，转移到```ProbeBW```
  - [ ] 转移到CRUISE, 避免ProbeRTT后增加inflight导致丢包?
  + 否则，转移到```Startup```

### 设计原因
+ ```ProbeRTTDuration=200ms```，足够长使不同RTT的流有重叠的```ProbeRTT```阶段，同时足够短使得浪费的吞吐量不超过2%
+ ```ProbeRTTInterval=5s```，足够短使快速适应链路的变化，同时足够长使得交互式应用常常会在5s内有低传输率的阶段，**该阶段会被捕捉**，从而不需要进行```ProbeRTT```即可更新```BBR.min_rtt```

### 代码
```
BBRUpdateMinRTT()
    BBR.probe_rtt_expired =
      Now() > BBR.probe_rtt_min_stamp + ProbeRTTInterval
    if (rs.rtt >= 0 and
        (rs.rtt < BBR.probe_rtt_min_delay or
         BBR.probe_rtt_expired))
       BBR.probe_rtt_min_delay = rs.rtt
       BBR.probe_rtt_min_stamp = Now()
    min_rtt_expired =
      Now() > BBR.min_rtt_stamp + MinRTTFilterLen
    if (BBR.probe_rtt_min_delay < BBR.min_rtt or
        min_rtt_expired)
      BBR.min_rtt       = BBR.probe_rtt_min_delay
      BBR.min_rtt_stamp = BBR.probe_rtt_min_stamp
```
---
```
BBRCheckProbeRTT():
    if (BBR.state != ProbeRTT and
        BBR.probe_rtt_expired and
        not BBR.idle_restart)
      BBREnterProbeRTT()
      BBRSaveCwnd()
      BBR.probe_rtt_done_stamp = 0
      BBR.ack_phase = ACKS_PROBE_STOPPING
      BBRStartRound()
    if (BBR.state == ProbeRTT)
      BBRHandleProbeRTT()
    if (rs.delivered > 0)
      BBR.idle_restart = false
  BBREnterProbeRTT():
    BBR.state = ProbeRTT
    BBR.pacing_gain = 1
    BBR.cwnd_gain = BBRProbeRTTCwndGain  /* 0.5 */
  BBRHandleProbeRTT():
    /* Ignore low rate samples during ProbeRTT: */
    MarkConnectionAppLimited()
    if (BBR.probe_rtt_done_stamp == 0 and
        packets_in_flight <= BBRProbeRTTCwnd())
      /* Wait for at least ProbeRTTDuration to elapse: */
      BBR.probe_rtt_done_stamp =
        Now() + ProbeRTTDuration
      /* Wait for at least one round to elapse: */
      BBR.probe_rtt_round_done = false
      BBRStartRound()
    else if (BBR.probe_rtt_done_stamp != 0)
      if (BBR.round_start)
        BBR.probe_rtt_round_done = true
      if (BBR.probe_rtt_round_done)
        BBRCheckProbeRTTDone()
  BBRCheckProbeRTTDone():
    if (BBR.probe_rtt_done_stamp != 0 and
        Now() > BBR.probe_rtt_done_stamp)
      /* schedule next ProbeRTT: */
      BBR.probe_rtt_min_stamp = Now()
      BBRRestoreCwnd()
      BBRExitProbeRTT()
  MarkConnectionAppLimited():
    C.app_limited =
      (C.delivered + packets_in_flight) ? : 1
```
- [ ] 在```BBR.state == ProbeRTT```时```BBR.probe_rtt_min_delay```的更新逻辑存在问题？
