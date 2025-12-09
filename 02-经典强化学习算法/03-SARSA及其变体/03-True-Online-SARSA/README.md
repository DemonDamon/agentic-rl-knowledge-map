---

# 2.3.3 True Online SARSA(λ)

---

## 1. 传统 SARSA(λ) 的问题 (Issues with Conventional SARSA(λ))

传统的、基于后向视图的 SARSA(λ) 算法（如上一节所述）虽然在理论上与前向视图的 λ-回报等价，但在实际的在线学习过程中存在一个微妙的延迟。这导致了所谓的“在线”学习并非“真正”的在线。

问题出在更新的时刻。在时间步 $t$，我们观察到 $R_{t+1}$ 和 $S_{t+1}$，并选择了 $A_{t+1}$。此时，我们计算出 TD 误差 $\delta_t = R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)$。然后，我们使用这个 $\delta_t$ 和在 $t$ 时刻的资格迹 $E_t$ 来更新 Q 函数。

然而，我们用来计算 $\delta_t$ 的 $Q(S_t, A_t)$ 是在 $t-1$ 时刻或更早之前计算的值。在 $t$ 时刻，当 $\delta_{t-1}$ 产生后，我们本应该用它和 $E_{t-1}$ 来更新 $Q(S_t, A_t)$。但这个更新被推迟到了 $t+1$ 时刻才与其他 Q 值一起进行。这种延迟意味着我们用来计算当前 TD 误差的 Q 值是一个“陈旧”的值，没有包含最新的信息。

这种延迟虽然不影响算法的收敛性，但会降低学习效率，并且使其在理论上不够优雅。True Online SARSA(λ) 的提出正是为了解决这个问题 [1]。

## 2. True Online SARSA(λ) 的思想 (The Idea of True Online SARSA(λ))

True Online SARSA(λ) 的目标是实现一个真正意义上的在线更新，使得算法在每一步都完全等价于理想的前向视图 λ-回报学习。

其核心思想是对传统的资格迹更新进行修正，引入一个“dutch trace”的变体，并对 Q 函数的更新方式进行巧妙的调整。

让我们回顾一下 SARSA(λ) 的更新：
$$ 
Q_{t+1}(s, a) = Q_t(s, a) + \alpha \delta_t E_t(s, a) 
$$
其中 $\delta_t = R_{t+1} + \gamma Q_t(S_{t+1}, A_{t+1}) - Q_t(S_t, A_t)$。

True Online SARSA(λ) 的关键洞察是，在计算 $\delta_t$ 时，我们应该使用“最新”的 Q 值，即 $Q_{t}(S_t, A_t)$ 应该是在 $t$ 时刻刚刚被 $\delta_{t-1}$ 更新过的值。为了实现这一点，算法将 Q 函数的更新分解为两部分。

在时间步 $t$，我们有旧的 Q 值 $Q_{t-1}$ 和资格迹 $E_{t-1}$。我们首先计算出当前状态-动作对 $(S_t, A_t)$ 的值 $Q_t(S_t, A_t)$，它等于旧值 $Q_{t-1}(S_t, A_t)$ 加上由于历史 TD 误差（通过资格迹 $E_{t-1}$ 传播）带来的增量。

True Online SARSA(λ) 的更新可以被推导为：

1.  **计算当前 Q 值的变化量**: 
    $$ 
    Q_{old} = Q(S_t, A_t) 
    $$

2.  **更新资格迹 (Dutch Trace)**:
    $$ 
    E(s, a) \leftarrow \gamma \lambda E(s, a) 
    $$
    $$ 
    E(S_t, A_t) \leftarrow E(S_t, A_t) + (1 - \alpha \gamma \lambda E(S_t, A_t)) 
    $$

3.  **计算 TD 误差**: 注意，这里用的是更新前的 $Q(S_t, A_t)$。
    $$ 
    \delta_t = R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) 
    $$

4.  **更新所有 Q 值**:
    $$ 
    Q(s, a) \leftarrow Q(s, a) + \alpha (\delta_t + Q(S_t, A_t) - Q_{old}) E(s, a) 
    $$
    $$ 
    Q(S_t, A_t) \leftarrow Q(S_t, A_t) - \alpha (Q(S_t, A_t) - Q_{old}) 
    $$

这个更新看起来比传统 SARSA(λ) 更复杂，但它在计算上是高效的，并且完美地实现了前向视图的语义。

## 3. 算法伪代码 (Pseudocode)

以下是 True Online SARSA(λ) 的简化版伪代码，它更清晰地揭示了其核心操作。

```
Algorithm: True Online SARSA(λ)

Initialize Q(s, a) arbitrarily, for all s, a
Initialize α, γ, λ, ε

Loop for each episode:
  Initialize S, A (chosen using policy from Q, e.g., ε-greedy)
  E = 0 (a scalar eligibility trace for the current (S,A) pair)
  Q_old = 0
  
  Loop for each step of episode:
    Take action A, observe R, S'
    Choose A' from S' using policy from Q (e.g., ε-greedy)
    
    Q_new = Q(S', A')
    δ = R + γ * Q_new - Q(S, A)
    
    E = γ * λ * E + 1
    
    // Update Q-values
    Q(S, A) = Q(S, A) + α * δ * E
    
    S ← S'; A ← A'
  until S is terminal
```
*注意：这是一个更易于理解的简化版本，实际实现需要更仔细地处理资格迹向量。*

完整的算法涉及到对资格迹向量的稀疏更新，以保持计算效率。

## 4. 优势 (Advantages)

-   **更高的样本效率**: 通过消除更新延迟，True Online SARSA(λ) 通常比传统 SARSA(λ) 学习得更快。
-   **更好的理论性质**: 它在每一步都精确地实现了与前向视图 λ-回报的等价性，使其在理论上更加完备。
-   **与函数逼近的良好结合**: True Online TD(λ) 的思想可以很好地与线性函数逼近结合，形成了高效的梯度 TD 方法。

由于这些优势，True Online SARSA(λ) 及其变体被认为是实现资格迹的现代标准方法。

## 5. 参考文献 (References)

1.  van Seijen, H., & Sutton, R. S. (2014, June). True online TD(λ). In *International conference on machine learning* (pp. 692-700).
2.  Sutton, R. S., & Barto, A. G. (2018). *Reinforcement learning: An introduction*. MIT press. (Section 12.8)
