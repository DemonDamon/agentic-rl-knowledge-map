worker" 智能体在各自的环境副本中收集经验。在每个固定的时间步（例如，每收集 N 步数据后），所有 worker 会暂停，并将它们收集到的经验数据汇总起来。

然后，一个中心化的学习器（通常在 GPU 上）使用这些汇总的数据来计算一次梯度更新，并同时更新演员和评论家的网络参数。更新完成后，新的网络参数被分发回所有的 worker，然后它们继续进行下一轮的经验收集。

**算法流程:**

1.  **初始化**: 初始化全局的演员网络参数 $\theta$ 和评论家网络参数 $w$。创建 N 个并行的 worker。
2.  **并行收集**: 每个 worker 从全局网络同步最新的参数，然后在自己的环境中执行 N 步，收集一个轨迹 $(S_0, A_0, R_1, \dots, S_{N-1}, A_{N-1}, R_N, S_N)$。
3.  **计算回报和优势**: 在每个 worker 收集完 N 步数据后，计算 N-步回报和优势函数。例如，使用 GAE：
    -   从最后一步开始，计算 $R_N = V_w(S_N)$ (如果 $S_N$ 不是终止状态)。
    -   从 $t = N-1$ 到 $0$ 循环，计算优势 $\hat{A}_t$ 和 N-步回报 $G_t = \hat{A}_t + V_w(S_t)$。
4.  **梯度计算**: 所有 worker 将它们的梯度汇总（或将数据汇总后由中心学习器计算梯度）。
    -   **评论家损失 (Critic Loss)**: $L_w = \frac{1}{N} \sum_t (G_t - V_w(S_t))^2$
    -   **演员损失 (Actor Loss)**: $L_\theta = -\frac{1}{N} \sum_t \hat{A}_t \log \pi_\theta(A_t|S_t)$
    -   (可选) **熵正则化 (Entropy Regularization)**: 为了鼓励探索，通常会加上一个熵奖励项 $H(\pi_\theta(S_t))$。
5.  **参数更新**: 中心学习器使用汇总的梯度来更新全局网络参数 $\theta$ 和 $w$。
6.  **循环**: 重复步骤 2-5。

## 4. 与 A3C 的比较 (Comparison with A3C)

-   **A3C (Asynchronous)**: 每个 worker 独立地计算梯度并**异步**地更新全局网络。这不需要一个中心协调器，但可能导致 worker 使用陈旧的策略，并且在 GPU 利用上效率较低。
-   **A2C (Synchronous)**: 所有 worker **同步**地收集数据和更新参数。这能更有效地利用 GPU 的并行计算能力，并且通常能使用更大的批量大小 (batch size)，从而获得更稳定和高效的学习。

在现代硬件上，A2C 通常比 A3C 更受欢迎。

## 5. 参考文献 (References)

1.  Schulman, J., Moritz, P., Levine, S., Jordan, M., & Abbeel, P. (2015). High-dimensional continuous control using generalized advantage estimation. *arXiv preprint arXiv:1506.02438*.
2.  Mnih, V., Badia, A. P., Mirza, M., Graves, A., Lillicrap, T., Harley, T., ... & Kavukcuoglu, K. (2016, June). Asynchronous methods for deep reinforcement learning. In *International conference on machine learning* (pp. 1928-1937). (This paper introduced A3C, and A2C is its synchronous counterpart).
3.  Wu, Y., Mansimov, E., Grosse, R. B., & Ba, J. (2017). Scalable trust-region method for deep reinforcement learning using Kronecker-factored approximation. In *Advances in neural information processing systems*.
