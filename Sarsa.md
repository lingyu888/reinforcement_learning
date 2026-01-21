* 等待 ChatGPT 润色

# Sarsa 算法与 Value Iteration 比较

## Sarsa

- 在 **Sarsa** 算法中，往往采用 **epsilon-greedy** 策略来平衡 **探索（exploration）** 和 **利用（exploitation）**。这就像是一棵树，偶尔会伸出一些枝桠去探索，但核心方向还是向上生长。这在 **Cliff Walking** 环境中体现为，智能体只找到了最优路径上的 **max_action**，导致只找到了第二排路径上的 **max_action**。具体体现如下：

```bash
# epsilon = 0.1
ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ovoo ooo> ooo> ovoo 
^ooo ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ovoo 
^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ooo> ^ooo ooo> ovoo 
^ooo **** **** **** **** **** **** **** **** **** **** EEEE 
```

- 同时可以观察到，第三排一开始远离了悬崖，这符合 Sarsa 无模型的特点，但接近终点时开始混乱，因为已经在第二排路径上找到了最优策略，导致第三排并没有被有效探索。

- 接着我修改了**epsilon**的大小，并考虑了边界条件。当**epsilon = 0**时，最优路径变为贴着悬崖走，这与**ValueIteration**中的最优路径结果一致，且**return**值稳定不变。也就是说，epsilon会影响探索和利用的平衡，当**epsilon = 0**时，智能体只利用当前策略，每次都选择max_action。

```bash
# epsilon = 0.0
^ooo ovoo ovoo ovoo ooo> ovoo ovoo ooo> ooo> ooo> ovoo ovoo 
ooo> ooo> ^ooo ooo> ooo> ovoo ooo> ooo> ovoo ^ooo ooo> ovoo 
ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ovoo 
^ooo **** **** **** **** **** **** **** **** **** **** EEEE 
```

- 当 **epsilon = 0.2** 时，最优路径远离悬崖，意味着较大的 **epsilon** 会使策略变得更加保守。这在 **return** 的计算公式中得以体现。我们可以理解为：**return = 0.2 * 悬崖的惩罚 + 0.8 * 接近终点的奖励**。当 **epsilon** 越大时，惩罚项所占比重升高，策略会倾向于远离悬崖。

```bash
# epsilon = 0.2
ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ovoo 
^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ooo> ooo> ovoo 
^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ^ooo ooo> ^ooo ooo> ovoo 
^ooo **** **** **** **** **** **** **** **** **** **** EEEE 
```

## Sarsa vs Value Iteration

- 进一步比较 **Sarsa** 和 **Value Iteration**，**Sarsa** 会找到最优路径，但最优路径外的点不再是最优的，而 **Value Iteration** 会找到所有点的 **max_action**，并且会标注出 **max_action** 的可能 **action**。这体现了 **Value Iteration** 是一个有模型的算法，直接利用 **贝尔曼方程** 求得所有 **state** 的 **value**。而 **Sarsa** 是无模型的，它只能依靠 **经验数据** 来求解已经探索过的位置。

- 通过对比，我们可以看到，当 **epsilon = 0.1** 时，第 2 排为最优路径，这需要先经历第 3 排获得数据，因此第 3 排的状态也相对较为稳定地选择 **max_action**，而第一排未被有效探索，因此其策略相对混乱。

```bash
# Value-Iteration
ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovoo 
ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovo> ovoo 
ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ooo> ovoo 
^ooo **** **** **** **** **** **** **** **** **** **** EEEE 
```

总结

Sarsa 算法：

采用 ε-greedy 策略，利用 epsilon 来控制 探索 和 利用。

当 ε = 0 时，智能体选择 最优路径，避免悬崖，但 最优路径外的状态 可能没有得到有效探索。

较大 ε 会增加 探索性，使得智能体避免过于冒险的路径，倾向于选择远离悬崖的路径。

Value Iteration：

采用 贝尔曼方程，通过 全局信息 来更新每个状态的 最优价值，保证所有状态的 最优动作 都能被找到。

不依赖于实际的交互过程，而是通过 全局更新 来计算最优策略。

主要区别：

Sarsa 是 on-policy 算法，策略的更新依赖于智能体实际执行的动作。它的学习过程受到 探索性（ε） 的影响，容易陷入局部最优。

Value Iteration 是 off-policy 算法，通过 全局模型 来计算最优策略，能够得到 全局最优解。