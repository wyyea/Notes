# 概率法
![[Pasted image 20240621014504.png]]
# 极值图论
## Mantel Theorem
![[Pasted image 20240620234347.png]]
无三角形图的边数的上界，利用数学归纳法反证，或者考虑$\Sigma(d_u + d_v)\leq n|E|$, 使用柯西不等式证明
## Turan's Theorem
![[Pasted image 20240620234443.png]]
无r-clique图的边数的上界，Mantel的一种推广
对含有最大边数的图，分离出r-clique，归纳证明
## Forbidden cycles
![[Pasted image 20240620235012.png]]
最小环长$\geq$ 5，边数的上界
对每个点，分析其邻居及邻居的邻居
## 

# 极值集合论

## Erdos-Rado(Sunflower Lemma)
![[Pasted image 20240620233614.png]]
数学归纳法证明
## Erdos-Ko-Rado theorem
![[Pasted image 20240620233652.png]]
任何相交k元集合族的大小的上界
## Sperner系统
![[Pasted image 20240620234024.png]]
一个集合族，若为反链，其大小的上界

证明：对$\mathcal{F}$中每个元素可能prefix的permutation总数进行计算
![[Pasted image 20240620234246.png]]
# Ramsey 理论
![[Pasted image 20240621002228.png]]
二着色
$R(k, l)\leq R(k-1, l) + R(k, l-1)$
![[Pasted image 20240621002343.png]]
多着色，混色理论
![[Pasted image 20240621002422.png]]
二着色的界
![[Pasted image 20240621002622.png]]
高维边，多着色
证明：先用混色降为二着色，再讨论高维边二着色
# 匹配论
## Hall's theorem
![[Pasted image 20240621003732.png]]
## Konig-Egervary theorem
![[Pasted image 20240621003817.png]]
二分图最大匹配与最小点覆盖的大小相等
## Dirworth's theorem
![[Pasted image 20240621003903.png]]
最大反链和最小链划分的大小相等