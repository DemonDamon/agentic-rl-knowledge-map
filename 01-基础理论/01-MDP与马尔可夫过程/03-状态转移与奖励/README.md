---

# 1.1.3 状态转移与奖励 (State Transitions and Rewards)

---

在马尔可夫决策过程 (MDP) 的五元组 $(\mathcal{S}, \mathcal{A}, P, R, \gamma)$ 定义中，状态转移概率函数 $P$ 和奖励函数 $R$ 共同构成了环境的**动态模型 (dynamics model)**。这个模型完整地描述了环境如何响应智能体的动作。

## 1. 状态转移概率函数 (State Transition Probability Function)

状态转移概率函数 $P$ 描述了环境的随机性。它指定了在给定当前状态和智能体动作后，环境将如何演化到下一个状态。

> **定义：状态转移概率**
>
> 状态转移概率函数 $P: \mathcal{S} \times \mathcal{A} \times \mathcal{S} \to [0, 1]$ 定义了从每个状态 $s \in \mathcal{S}$ 执行动作 $a \in \mathcal{A}(s)$ 后，转移到状态 $s' \in \mathcal{S}$ 的概率：
> $$ 
> P(s'|s, a) \triangleq \mathbb{P}(S_{t+1} = s' | S_t = s, A_t = a) 
> $$

这个函数必须满足马尔可夫性质，即下一状态的分布仅依赖于当前的状态和动作。同时，作为一个概率分布，它必须满足：
$$ 
\forall s \in \mathcal{S}, a \in \mathcal{A}(s), \quad \sum_{s' \in \mathcal{S}} P(s'|s, a) = 1 
$$

在模型驱动 (model-based) 的强化学习中，我们假设 $P$ 是已知的。而在无模型 (model-free) 的设定下，$P$ 是未知的，智能体必须通过与环境的交互来间接学习其影响。

## 2. 奖励函数 (Reward Function)

奖励函数 $R$ 定义了智能体在特定交互步骤中的目标。它是智能体行为好坏的即时反馈信号。强化学习的目标是最大化累积奖励，而非即时奖励。

奖励函数最通用的形式是依赖于当前状态 $s$、采取的动作 $a$ 以及转移到的下一状态 $s'$：

> **定义：奖励函数**
>
> 奖励函数 $R: \mathcal{S} \times \mathcal{A} \times \mathcal{S} \to \mathbb{R}$ 指定了在状态 $s$ 执行动作 $a$ 并转移到状态 $s'$ 后，智能体接收到的期望奖励：
> $$ 
> R(s, a, s') \triangleq \mathbb{E}[R_{t+1} | S_t = s, A_t = a, S_{t+1} = s'] 
> $$

为了简化表示，奖励函数常常被定义为状态-动作对的函数 $R: \mathcal{S} \times \mathcal{A} \to \mathbb{R}$，它表示执行一个动作的期望即时奖励，通过对所有可能的下一状态进行积分或求和得到：
$$ 
_R(s, a) \triangleq \mathbb{E}[R_{t+1} | S_t = s, A_t = a] = \sum_{s' \in \mathcal{S}} P(s'|s, a) R(s, a, s') 
$$

## 3. 环境模型 (The Environment Model)

状态转移函数 $P$ 和奖励函数 $R$ 一起，构成了对环境的完整描述。给定任何状态 $s$ 和动作 $a$，这个模型能够给出所有可能的下一状态 $s'$ 和奖励 $r$ 的联合概率分布 $p(s', r | s, a)$：
$$ 
_p(s', r | s, a) \triangleq \mathbb{P}(S_{t+1}=s', R_{t+1}=r | S_t=s, A_t=a) 
$$

从这个联合分布中，我们可以重新推导出状态转移概率和期望奖励：
-   **状态转移概率**: $P(s'|s, a) = \sum_{r \in \mathcal{R}} p(s', r | s, a)$
-   **期望奖励**: $R(s, a) = \sum_{r \in \mathcal{R}} r \sum_{s' \in \mathcal{S}} p(s', r | s, a)$

### 示例：简单的网格世界 (Simple Gridworld Example)

考虑一个 $3 \times 1$ 的网格世界，状态空间 $\mathcal{S} = \{s_1, s_2, s_3\}$，其中 $s_1$ 是起始状态，$s_3$ 是终止状态。动作空间 $\mathcal{A} = \{\text{left}, \text{right}\}$。

-   **状态转移**: 在 $s_1$ 执行 `right` 确定性地到达 $s_2$。在 $s_2$ 执行 `right` 确定性地到达 $s_3$。在 $s_2$ 执行 `left` 确定性地回到 $s_1$。其他动作（如在 $s_1$ 执行 `left`）则保持状态不变。
    -   $P(s_2 | s_1, \text{right}) = 1$
    -   $P(s_1 | s_1, \text{left}) = 1$
    -   $P(s_3 | s_2, \text{right}) = 1$
    -   $P(s_1 | s_2, \text{left}) = 1$

-   **奖励函数**: 到达终止状态 $s_3$ 获得 +1 奖励，其他所有转移的奖励均为 0。
    -   $R(s_2, \text{right}, s_3) = +1$
    -   所有其他的 $R(s, a, s') = 0$

这个简单的模型完整地定义了智能体将要面对的环境，所有强化学习算法的目标都是在这个模型（无论已知或未知）中寻找最优的行为策略。

## 4. 参考文献 (References)

1.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 3.2)
2.  Bellman, R. (1957). A Markovian decision process. *Journal of Mathematics and Mechanics*, 673-684.
