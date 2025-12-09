---

# 1.2.3.2 策略迭代收敛性证明 (Proof of Convergence for Policy Iteration)

---

## 1. 证明逻辑 (Logic of the Proof)

策略迭代算法的收敛性证明依赖于两个关键的理论基石：

1.  **策略改进定理 (Policy Improvement Theorem):** 该定理保证了由贪婪化生成的新策略 $\pi_{k+1}$ 的值函数 $V^{\pi_{k+1}}$ 必然不劣于原策略 $\pi_k$ 的值函数 $V^{\pi_k}$，即 $V^{\pi_{k+1}}(s) \geq V^{\pi_k}(s)$ 对所有状态 $s$ 成立。

2.  **有限策略数量 (Finite Number of Policies):** 对于一个具有有限状态空间 $|\mathcal{S}|$ 和有限动作空间 $|\mathcal{A}|$ 的马尔可夫决策过程 (MDP)，确定性策略的总数是有限的，其上界为 $|\mathcal{A}|^{|\mathcal{S}|}$。

证明的核心逻辑是：策略迭代算法在每一步都会产生一个不差于前一步的策略。如果某一步的改进是严格的（即新策略严格优于旧策略），那么值函数就会严格增加。由于在一个有限 MDP 中，不同的策略数量是有限的，而每个策略对应一个唯一的值函数，因此这个严格递增的过程不可能无限持续下去。它最终必然会达到一个点，使得策略无法再被改进，此时算法收敛。

## 2. 详细证明 (Detailed Proof)

令策略迭代过程生成的策略序列为 $\pi_0, \pi_1, \pi_2, \dots$。

在第 $k$ 次迭代中，我们首先进行**策略评估**，计算出 $V^{\pi_k}$。然后进行**策略改进**，生成新策略 $\pi_{k+1}$：
$$ 
\pi_{k+1}(s) = \arg\max_{a \in \mathcal{A}} Q^{\pi_k}(s, a) 
$$

根据策略改进定理，我们知道 $V^{\pi_{k+1}}(s) \geq V^{\pi_k}(s)$ 对所有 $s \in \mathcal{S}$ 成立。

现在，我们考虑算法的终止条件。算法在策略变得稳定时终止，即 $\pi_{k+1} = \pi_k$。

假设在第 $k$ 步，策略变得稳定，即 $\pi_{k+1}(s) = \pi_k(s)$ 对所有 $s$ 成立。这意味着对于所有 $s$：
$$ 
\pi_k(s) = \arg\max_{a \in \mathcal{A}} Q^{\pi_k}(s, a) 
$$

将其展开，我们得到：
$$ 
Q^{\pi_k}(s, \pi_k(s)) = \max_{a \in \mathcal{A}} Q^{\pi_k}(s, a) 
$$

根据状态值函数和动作值函数的关系 $V^{\pi_k}(s) = Q^{\pi_k}(s, \pi_k(s))$，上式可以写为：
$$ 
V^{\pi_k}(s) = \max_{a \in \mathcal{A}} Q^{\pi_k}(s, a) 
$$

将 $Q^{\pi_k}(s, a)$ 的定义代入：
$$ 
V^{\pi_k}(s) = \max_{a \in \mathcal{A}} \left( R(s, a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s, a) V^{\pi_k}(s') \right) 
$$

我们发现，此时的 $V^{\pi_k}$ 已经满足了**贝尔曼最优性方程**。根据贝尔曼最优性方程的唯一解性质，我们必然有 $V^{\pi_k} = V^*$。因此，策略 $\pi_k$ 就是一个最优策略 $\pi^*$。

**那么，这个过程是否保证会终止呢？**

假设算法不终止。这意味着策略序列会无限地进行严格改进，即 $V^{\pi_{k+1}}(s) > V^{\pi_k}(s)$ 在至少一个状态上成立，并且对于所有状态 $V^{\pi_{k+1}}(s) \geq V^{\pi_k}(s)$。这意味着每一个策略 $\pi_k$ 都与之前的策略 $\pi_0, \dots, \pi_{k-1}$ 不同。

然而，在一个有限 MDP 中，确定性策略的总数是有限的，最多为 $|\mathcal{A}|^{|\mathcal{S}|}$。因此，一个由不同策略组成的序列的长度不可能是无限的。这个矛盾证明了我们的假设（算法不终止）是错误的。

因此，策略迭代算法必然会在有限次迭代内收敛。由于收敛时得到的策略满足贝尔曼最优性方程，所以该算法保证收敛到最优策略。

## 3. 收敛速度 (Rate of Convergence)

策略迭代通常收敛得非常快，在实践中往往只需要很少的几次迭代就能找到最优策略。尽管其最坏情况下的迭代次数是策略的总数，但这在实际中极少发生。其主要的计算瓶颈在于每次迭代中的策略评估步骤，该步骤本身可能需要大量的计算来使值函数收敛。

## 4. 参考文献 (References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 4.3)
2.  Puterman, M. L. (2014). *Markov decision processes: discrete stochastic dynamic programming*. John Wiley & Sons. (Section 6.3)
