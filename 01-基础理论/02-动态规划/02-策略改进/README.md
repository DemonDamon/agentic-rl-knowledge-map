---

# 1.2.2 策略改进 (Policy Improvement)

---

## 1. 原理 (Principle)

策略评估的目的是为了回答“当前策略 $\pi$ 有多好？”。一旦我们通过计算得到了其值函数 $V^\pi$，一个很自然的问题便是：“我们能否找到一个更好的策略？”

策略改进的原理就是基于当前策略的值函数，通过贪婪地选择动作来构建一个新策略 $\pi'$，并保证这个新策略不差于（通常优于）原始策略 $\pi$。

具体来说，对于某个状态 $s$，我们已经知道了遵循策略 $\pi$ 的价值 $V^\pi(s)$。现在，我们考虑在状态 $s$ 选择一个不同于 $\pi(a|s)$ 所规定的动作 $a$，而在所有后续状态 $s'$ 中仍然遵循策略 $\pi$。那么，采取这个动作 $a$ 的价值是多少呢？这正是动作值函数 $Q^\pi(s, a)$ 的定义：
$$ 
Q^\pi(s, a) \triangleq \mathbb{E}[R_{t+1} + \gamma V^\pi(S_{t+1}) | S_t = s, A_t = a] = \sum_{s'} P(s'|s,a) [R(s,a,s') + \gamma V^\pi(s')] 
$$

如果我们在状态 $s$ 发现存在某个动作 $a$ 使得 $Q^\pi(s, a) > V^\pi(s)$，这就意味着在状态 $s$ 选择动作 $a$，然后继续遵循策略 $\pi$，会比从一开始就完全遵循策略 $\pi$ 得到更高的期望回报。这启发我们可以通过选择具有最大 $Q$ 值的动作来改进策略。

## 2. 策略改进定理 (Policy Improvement Theorem)

策略改进定理为上述思想提供了坚实的理论保证。它断言，通过对原策略 $\pi$ 进行贪婪化得到的新策略 $\pi'$ 一定是一个不差于 $\pi$ 的策略。

> **定理：策略改进定理**
>
> 令 $\pi$ 和 $\pi'$ 为任意两个确定性策略，如果对于所有状态 $s \in \mathcal{S}$，它们满足：
> $$ 
> Q^\pi(s, \pi'(s)) \geq V^\pi(s) 
> $$ 
> 那么策略 $\pi'$ 必然不差于策略 $\pi$，即对于所有状态 $s \in \mathcal{S}$，都有：
> $$ 
> V^{\pi'}(s) \geq V^\pi(s) 
> $$ 

**证明**: 
从 $V^\pi(s)$ 的定义开始，我们可以进行一系列的展开和代换：
$$ 
\begin{aligned} 
V^\pi(s) & \leq Q^\pi(s, \pi'(s)) & \text{(根据前提条件)} \\ 
& = \mathbb{E}_{\pi'}[R_{t+1} + \gamma V^\pi(S_{t+1}) | S_t = s] & \text{(将Q函数展开)} \\ 
& \leq \mathbb{E}_{\pi'}[R_{t+1} + \gamma Q^\pi(S_{t+1}, \pi'(S_{t+1})) | S_t = s] & \text{(再次应用前提条件于 } S_{t+1}) \\ 
& = \mathbb{E}_{\pi'}[R_{t+1} + \gamma \mathbb{E}_{\pi'}[R_{t+2} + \gamma V^\pi(S_{t+2}) | S_{t+1}] | S_t = s] & \text{(再次展开Q函数)} \\ 
& = \mathbb{E}_{\pi'}[R_{t+1} + \gamma R_{t+2} + \gamma^2 V^\pi(S_{t+2}) | S_t = s] & \text{(期望的性质)} \\ 
& \vdots & \\ 
& \leq \mathbb{E}_{\pi'}[R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \dots | S_t = s] & \text{(重复展开)} \\ 
& = V^{\pi'}(s) 
\end{aligned} 
$$
因此，得证 $V^{\pi'}(s) \geq V^\pi(s)$。

如果不等式 $Q^\pi(s, \pi'(s)) > V^\pi(s)$ 在至少一个状态上严格成立，那么 $V^{\pi'}(s) > V^\pi(s)$ 也将在至少一个状态上严格成立（除非这些状态在策略 $\pi'$ 下永远无法达到）。

## 3. 贪婪策略 (Greedy Policy)

基于策略改进定理，我们可以通过对当前值函数 $V^\pi$ 采取贪婪化的方式来构建一个新的、改进的策略 $\pi'$。

$$ 
\begin{aligned} 
\pi'(s) & = \arg\max_{a \in \mathcal{A}} Q^\pi(s, a) \\ 
& = \arg\max_{a \in \mathcal{A}} \left\{ R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V^\pi(s') \right\} 
\end{aligned} 
$$

这个新的贪婪策略 $\pi'$ 通过在每个状态 $s$ 选择能够带来最大短期回报（加上后继状态的折扣价值）的动作，来改进原始策略 $\pi$。根据策略改进定理，我们知道 $V^{\pi'} \geq V^\pi$。

这个过程——先评估一个策略，然后通过贪婪化来改进它——是**策略迭代 (Policy Iteration)** 算法的核心步骤。如果 $V^{\pi'} = V^\pi$，那么说明策略已经无法再被改进，此时 $V^\pi$ 必然等于 $V^*$，而 $\pi$ 也必然是一个最优策略。这是因为此时 $V^\pi$ 已经满足了贝尔曼最优性方程：
$$ 
V^\pi(s) = V^{\pi'}(s) = \max_{a \in \mathcal{A}} Q^\pi(s, a) 
$$

## 4. 参考文献 (References)

1. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 4.2)
2. Howard, R. A. (1960). *Dynamic programming and Markov processes*. MIT press.
