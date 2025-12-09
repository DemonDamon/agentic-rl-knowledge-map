---

# 1.2.3.3 策略迭代复杂度分析 (Complexity Analysis of Policy Iteration)

---

## 1. 概述 (Overview)

策略迭代算法的计算复杂度主要由其两个核心步骤——策略评估和策略改进——决定。其总复杂度是这两部分复杂度与策略改进所需总迭代次数的乘积。我们将分别分析每个组成部分的时间复杂度和算法所需的空间复杂度。

假设一个有限马尔可夫决策过程 (MDP) 具有：
-   $|\mathcal{S}|$ 个状态
-   $|\mathcal{A}|$ 个动作

## 2. 时间复杂度 (Time Complexity)

策略迭代的总时间复杂度可以表示为：
$$ 
O(\text{Num_PI_Iterations} \times (\text{Complexity_of_PE} + \text{Complexity_of_PI})) 
$$
其中 `Num_PI_Iterations` 是策略改进所需的总迭代次数。

### 2.1 策略评估 (Policy Evaluation)

策略评估步骤通过迭代计算来求解一个线性方程组。在每次迭代中，都需要对所有状态进行一次完整的扫描 (sweep)。

-   对于每个状态 $s \in \mathcal{S}$，我们需要计算：
    $$ 
    V_{k+1}(s) \leftarrow \sum_{a \in \mathcal{A}} \pi(a|s) \left( R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V_k(s') \right) 
    $$
-   对于一个稠密的转移矩阵 $P$，计算 $\sum_{s'} P(s'|s,a) V_k(s')$ 需要 $O(|\mathcal{S}|)$ 次操作。
-   对于一个确定性策略 $\pi$，我们只需要考虑一个动作。因此，更新一个状态的值需要 $O(|\mathcal{S}|)$ 的时间。
-   如果策略是随机的，我们需要对所有动作求和，更新一个状态的值需要 $O(|\mathcal{A}||\mathcal{S}|)$ 的时间。
-   对所有 $|\mathcal{S}|$ 个状态进行一次完整的扫描，其计算复杂度为 $O(|\mathcal{S}|^2 |\mathcal{A}|)$。

策略评估的迭代次数依赖于折扣因子 $\gamma$ 和所需的精度 $\epsilon$。理论上，迭代次数可以界定为 $O(\log(1/\epsilon) / (1-\gamma))$。因此，一次完整的策略评估的计算复杂度为：
$$ 
O(N_{PE} \cdot |\mathcal{S}|^2 |\mathcal{A}|) 
$$
其中 $N_{PE}$ 是评估所需的迭代次数。

或者，我们也可以通过求解线性方程组 $(I - \gamma P^\pi)V^\pi = R^\pi$ 来直接计算 $V^\pi$。使用高斯消元法等直接求解器，其复杂度为 $O(|\mathcal{S}|^3)$。

### 2.2 策略改进 (Policy Improvement)

策略改进步骤需要为每个状态找到最优的动作。

-   对于每个状态 $s \in \mathcal{S}$，我们需要计算：
    $$ 
    \pi'(s) \leftarrow \arg\max_{a \in \mathcal{A}} \left( R(s,a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s,a) V^\pi(s') \right) 
    $$
-   对于每个动作 $a$，计算其 Q 值需要 $O(|\mathcal{S}|)$ 次操作。
-   我们需要对所有 $|\mathcal{A}|$ 个动作进行比较。
-   因此，改进一个状态的策略需要 $O(|\mathcal{A}||\mathcal{S}|)$ 的时间。
-   对所有 $|\mathcal{S}|$ 个状态进行一次完整的策略改进，其计算复杂度为：
    $$ 
    O(|\mathcal{S}|^2 |\mathcal{A}|) 
    $$

### 2.3 总体复杂度

策略迭代所需的总迭代次数 $N_{PI}$ 在理论上最多为所有可能策略的数量，但在实践中通常是一个很小的常数。

因此，使用迭代法进行策略评估的策略迭代算法，其每次主循环的复杂度为 $O(N_{PE} \cdot |\mathcal{S}|^2 |\mathcal{A}| + |\mathcal{S}|^2 |\mathcal{A}|)$。如果使用直接法求解，则为 $O(|\mathcal{S}|^3 + |\mathcal{S}|^2 |\mathcal{A}|)$。

## 3. 空间复杂度 (Space Complexity)

策略迭代算法需要存储以下信息：

1.  **状态值函数**: 需要一个数组来存储每个状态的值。对于迭代评估，可能需要两个数组（一个用于旧值，一个用于新值）来避免原地更新带来的问题。复杂度为 $O(|\mathcal{S}|)$。
2.  **策略**: 需要一个数组来存储每个状态对应的动作（对于确定性策略）。复杂度为 $O(|\mathcal{S}|)$。
3.  **环境模型**: 算法假设模型是已知的，因此需要存储状态转移概率 $P(s'|s,a)$ 和奖励函数 $R(s,a)$。在最坏情况下，这需要 $O(|\mathcal{S}|^2 |\mathcal{A}|)$ 的空间。

因此，总的空间复杂度主要由存储环境模型决定，为 $O(|\mathcal{S}|^2 |\mathcal{A}|)$。

## 4. 与值迭代的比较 (Comparison with Value Iteration)

| 特性 | 策略迭代 (Policy Iteration) | 值迭代 (Value Iteration) |
| :--- | :--- | :--- |
| **主循环** | 交替评估和改进 | 直接迭代贝尔曼最优性方程 |
| **内循环** | 包含一个完整的、多步的策略评估循环 | 无内循环，每次迭代是单次扫描 |
| **迭代次数** | 主循环迭代次数少，但每次迭代成本高 | 主循环迭代次数多，但每次迭代成本低 |
| **复杂度/次迭代** | $O(|\mathcal{S}|^3)$ 或 $O(N_{PE} \cdot |\mathcal{S}|^2 |\mathcal{A}|)$ | $O(|\mathcal{S}|^2 |\mathcal{A}|)$ |
| **收敛性** | 策略在有限步内收敛到最优 | 值函数渐近收敛到最优 |
| **适用场景** | 当策略空间远小于状态空间时可能更快 | 当状态空间巨大时通常更高效 |

## 5. 参考文献 (References)

1.  Littman, M. L., Dean, T. L., & Kaelbling, L. P. (1995). On the complexity of solving Markov decision problems. In *Uncertainty in Artificial Intelligence* (pp. 394-402).
2.  Puterman, M. L. (2014). *Markov decision processes: discrete stochastic dynamic programming*. John Wiley & Sons. (Chapter 6)
