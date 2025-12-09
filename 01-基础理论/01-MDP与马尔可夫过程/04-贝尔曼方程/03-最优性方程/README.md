---

# 1.1.4.3 贝尔曼最优性方程 (Bellman Optimality Equation)

---

## 1. 概述 (Overview)

贝尔曼最优性方程描述了在所有可能的策略中，最优策略所能达到的价值函数。与贝尔曼期望方程描述特定策略的价值不同，最优性方程定义了强化学习问题的“解”——即最优值函数 $V^*(s)$ 和 $Q^*(s, a)$ [1]。

最优策略 $\pi^*$ 是指在所有状态下，其期望回报不劣于任何其他策略 $\pi$ 的策略，即 $\forall s \in \mathcal{S}, V^{\pi^*}(s) \geq V^\pi(s)$。

> **定义：最优状态值函数与最优动作值函数**
> $$ 
> V^*(s) \triangleq \max_{\pi} V^\pi(s) 
> $$ 
> $$ 
> Q^*(s, a) \triangleq \max_{\pi} Q^\pi(s, a) 
> $$

贝尔曼最优性方程的关键在于它包含了一个**最大化 (maximization)** 操作。它断言，最优策略下的价值，等于在当前状态/状态-动作对下，选择最优动作后所能达到的期望回报。这个 `max` 操作使得方程变为非线性，因此不能像线性方程组那样直接求解，但它为值迭代等动态规划算法提供了更新规则。

## 2. 最优状态值函数 $V^*(s)$ (Optimal State-Value Function)

一个状态的最优价值 $V^*(s)$，必然等于从该状态出发，选择当前所能采取的**最优动作**后，所能获得的期望回报。这个最优动作对应的价值由最优动作值函数 $Q^*(s, a)$ 给出。因此，我们有：
$$ 
V^*(s) = \max_{a \in \mathcal{A}(s)} Q^*(s, a) 
$$

接下来，我们将 $Q^*(s, a)$ 的定义展开。$Q^*(s, a)$ 是在状态 $s$ 执行动作 $a$ 后，再永远遵循最优策略所得到的期望回报。这等于即时奖励加上后继状态的最优价值的折扣期望：
$$ 
Q^*(s, a) = R(s, a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s, a) V^*(s') 
$$

将这两个关系式结合，我们就得到了**状态值函数 $V^*(s)$ 的贝尔曼最优性方程**：

> **贝尔曼最优性方程 (V-function)**
> $$ 
> V^*(s) = \max_{a \in \mathcal{A}} \left( R(s, a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s, a) V^*(s') \right) 
> $$

### 备份图 (Backup Diagram for $V^*$)

这个过程的备份图与 $V^\pi$ 的类似，但在第一层增加了一个 `max` 操作，表示从所有可能的动作中选择价值最大的那一个。

```mermaid
graph TD
    subgraph "V*(s)"
        S_t(s)
    end

    subgraph "Actions"
        A_t1((a1))
        A_t2((a2))
        A_tn((...))
    end

    subgraph "Successor States"
        S_t1_1((s"))
        S_t1_2((s" "))
        S_t2_1((s"" "))
        S_tn_1((...))
    end

    S_t -- max_a --> A_t1
    S_t -- max_a --> A_t2
    S_t -- max_a --> A_tn

    A_t1 -- P(s"|s,a1) --> S_t1_1
    A_t1 -- P(s" "|s,a1) --> S_t1_2
    A_t2 -- P(s"" "|s,a2) --> S_t2_1
    A_tn -- ... --> S_tn_1
```

## 3. 最优动作值函数 $Q^*(s, a)$ (Optimal Action-Value Function)

对于最优动作值函数 $Q^*(s, a)$，其推导更为直接。在状态 $s$ 执行动作 $a$ 之后，环境转移到下一状态 $s'$。为了保持最优性，智能体在状态 $s'$ 之后也必须遵循最优策略。因此，从 $s'$ 开始能获得的期望回报就是 $V^*(s')$。

所以，$Q^*(s, a)$ 等于执行动作 $a$ 的即时奖励，加上所有可能的下一状态 $s'$ 的最优价值的折扣期望：
$$ 
Q^*(s, a) = R(s, a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s, a) V^*(s') 
$$

我们知道 $V^*(s') = \max_{a'} Q^*(s', a')$。将其代入上式，就得到了**动作值函数 $Q^*(s, a)$ 的贝尔曼最优性方程**：

> **贝尔曼最优性方程 (Q-function)**
> $$ 
> Q^*(s, a) = R(s, a) + \gamma \sum_{s' \in \mathcal{S}} P(s'|s, a) \max_{a'} Q^*(s', a') 
> $$

这个方程是 Q-Learning 算法的理论基础。

### 备份图 (Backup Diagram for $Q^*$)

$Q^*$ 的备份图从一个状态-动作对 $(s, a)$ 开始，展望其所有可能的后继状态 $s'$，然后在每个 $s'$ 处，通过 `max` 操作选择最优的下一动作。

```mermaid
graph TD
    subgraph "Q*(s,a)"
        SA_t((s,a))
    end

    subgraph "Successor States"
        S_t1((s"))
        S_t2((s" "))
        S_tn((...))
    end

    subgraph "Optimal Successor Actions"
        SA_t1_1(((s", a"*)))
        SA_t2_1(((s" ", a""*)))
        SA_tn_1(((...)))
    end

    SA_t -- P(s"|s,a) --> S_t1
    SA_t -- P(s" "|s,a) --> S_t2
    SA_t -- ... --> S_tn

    S_t1 -- max_a" --> SA_t1_1
    S_t2 -- max_a"" --> SA_t2_1
    S_tn -- ... --> SA_tn_1
```

## 4. 参考文献 (References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 3.7)
2.  Bellman, R. (1957). A Markovian decision process. *Journal of Mathematics and Mechanics*, 673-684.
