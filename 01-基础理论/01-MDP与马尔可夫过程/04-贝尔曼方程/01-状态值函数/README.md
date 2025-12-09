 
        S_t1_1((s'))
        S_t1_2((s''))
        S_t2_1((s'''))
        S_tn_1((...))
    end

    S_t -- π(a1|s) --> A_t1
    S_t -- π(a2|s) --> A_t2
    S_t -- ... --> A_tn

    A_t1 -- P(s'|s,a1) --> S_t1_1
    A_t1 -- P(s''|s,a1) --> S_t1_2
    A_t2 -- P(s'''|s,a2) --> S_t2_1
    A_tn -- ... --> S_tn_1
```

-   顶层空心圆圈代表状态 $s$。
-   从 $s$ 出发的边代表根据策略 $\pi$ 选择的各个动作 $a$。
-   第二层的实心圆圈代表状态-动作对。
-   从每个状态-动作对出发的边代表根据环境动态 $P$ 可能转移到的各个下一状态 $s'$。
-   $V^\pi(s)$ 是对所有动作的期望，而每个动作的价值又是对所有下一状态的期望。

## 3. 矩阵形式 (Matrix Form)

对于一个有限状态 MDP，贝尔曼期望方程可以写成一个简洁的矩阵形式。令 $\mathbf{v}^\pi$ 是一个 $|\mathcal{S}| \times 1$ 的列向量，其中每个元素是 $V^\pi(s)$。

定义策略 $\pi$ 下的期望奖励向量 $\mathbf{r}^\pi$ 和状态转移矩阵 $\mathbf{P}^\pi$：
-   $(\mathbf{r}^\pi)_s = \sum_{a \in \mathcal{A}} \pi(a|s) R(s,a)$
-   $(\mathbf{P}^\pi)_{ss'} = \sum_{a \in \mathcal{A}} \pi(a|s) P(s'|s,a)$

那么，贝尔曼期望方程可以表示为：
$$ 
\mathbf{v}^\pi = \mathbf{r}^\pi + \gamma \mathbf{P}^\pi \mathbf{v}^\pi 
$$

这是一个标准的线性方程组。如果模型已知，原则上可以直接求解：
$$ 
(\mathbf{I} - \gamma \mathbf{P}^\pi) \mathbf{v}^\pi = \mathbf{r}^\pi \implies \mathbf{v}^\pi = (\mathbf{I} - \gamma \mathbf{P}^\pi)^{-1} \mathbf{r}^\pi 
$$
其中 $\mathbf{I}$ 是单位矩阵。矩阵 $(\mathbf{I} - \gamma \mathbf{P}^\pi)$ 的可逆性由 $\gamma < 1$ 保证。然而，由于矩阵求逆的计算复杂度为 $O(|\mathcal{S}|^3)$，对于大规模问题，通常采用迭代方法（如迭代策略评估）来求解。

## 4. 参考文献 (References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 3.5)
2.  Bellman, R. (1957). *Dynamic programming*. Princeton university press.
