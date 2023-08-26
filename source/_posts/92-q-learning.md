---
url: q-learning
title: Q-Learning 简介
date: 2022-04-05 10:50:56
categories: [技术]
tags: [机器学习, 强化学习, Q-Learning]
mathjax: true
---
强化学习（Reinforcement Learning）是一种用于模拟现实世界的算法，而其中的一个重要的组成部分是 Q-Learning。

<!--more-->

假设有一系列「状态」（state）组成的环境，在每一个状态中都可以采取一种「动作」（action）到达下一个状态。 这些状态并不都是相同的，根据实际应用，不同的状态有不同的「奖励」（reward），而且存在终止状态。我们的目标是从任意初始状态出发，到达终止状态，且所获得的奖励最大。

Q-Learning 就是这样一种决策算法，帮助我们在每个状态决定应该采取何种动作，才能最终得到最大奖励。

## 算法描述

Q-Learning 算法的核心是维护一张 Q 表，它的每一行对应一个状态，每一列对应一种动作，每个值代表了这个状态下采取这种动作最终可以得到的最大奖励。Q-Learning 会经过多次尝试去更新其中的值，不断试错，从而“学习”到外部环境的机制。

步骤如下：

1. 随机初始化 $Q(s, a)$
2. 重复以下步骤，每一次重复记作一次 episode：
	1. 初始化状态 s
	2. 重复以下步骤，每一次重复记作一次 episode 的一步（step）：
		1. 选择状态 s 下要采取的动作 a（依据一定的策略，如 $\epsilon$-greedy）
		2. 执行动作 a，得到奖励 r，到达下一状态 s'
		3. 更新 Q 表：$Q(s,a)\leftarrow Q(s,a)+\alpha[r+\gamma max_{a'}Q(s',a')-Q(s,a)]$
		4. 更新状态：$s\leftarrow s'$
	3. 当 s 为终止状态，停止

要说明的：

- 在每个 step 中，要首先根据一定策略选择要执行的动作。其中 $\epsilon$-greedy 策略是在一定概率下按 Q 表的最优值选择动作，其他情况下随机选择动作。
- 在更新 Q 表的步骤中，$r+\gamma max_{a'}Q(s',a')$ 是当前得到的最大奖励值（当前状态 s 采取动作 a 获得的奖励 + 下一状态 s' 获得的最大奖励（Q 表预测的 ）），而 $Q(s,a)$ 则是 Q 表预测的最大奖励值。将二者作差并乘上一个学习率 $\alpha$，叠加到现有的 Q 表中，就完成了 Q 表的一次更新。
- 上面中出现的 $\gamma$ 是衰减率，为 0 即不考虑未来奖励如何，只考虑当前奖励最大；为 1 即考虑所有未来的奖励；介于 0 和 1 之间则考虑的程度随 $\gamma^2$ 不断衰减。

## 代码演示

根据上面数学角度的分析，不难看出实际需要处理的就是两个部分：选择动作和更新 Q 表。下面用 `python` 演示这两部分。

### `__init__`

初始化函数主要为建立 Q 表：

```python
def __init__(self, actions, learning_rate=0.01, reward_decay=0.9, e_greedy=0.9):
    self.actions = actions
    self.lr = learning_rate
    self.gamma = reward_decay
    self.epsilon = e_greedy
    self.q_table = pd.DataFrame(columns=self.actions, dtype=np.float64)
```

其中 `actions` 为动作的集合（`list` 类型），需要用到 `pandas` 和 `numpy` 库。

### `choose_action`

`choose_action` 是选择动作的函数，需要接收一个参数作为当前状态：

```python
def choose_action(self, observation):
```

为方便后续处理和操作，将参数转换为 `str` 类型：

```python
observation = str(observation)
```

接下来需要选择要执行的动作了，但我们首先要判断参数对应的状态是否在 Q 表中，需要引入函数 `check_state_exist`：

```python
def check_state_exist(self, state):
    if state not in self.q_table.index:
        # append new state to q table
        self.q_table = self.q_table.append(
            pd.Series(
                [0] * len(self.actions),
                index=self.q_table.columns,
                name=state,
            )
        )
```

逻辑很简单，当这个状态不在 Q 表中时，我们就把它加入 Q 表。

{% note warning %}

若此处出现报错 `TypeError: unhashable type: 'list'`，应该是把 `list` 类型的 `state` 作为参数传了过来。`list` 类型是不可以作为 key 的，注意需要转换为 `str` 类型。

{% endnote %}

在 `choose_action` 中调用函数 `check_state_exist` 进行判断：

```python
self.check_state_exist(observation)
```

接下来就可以进行动作的选择了！

注意之前提到的 $\epsilon$-greedy 策略，我们需要在一定的概率下选择 Q 表中最优值：

```python
if np.random.uniform() < self.epsilon:
    # choose best action
else:
    # choose random action
```

回忆一下，Q 表每一行对应一个状态、每一列对应一种动作。所以在 Q 表中选择动作，首先要找到当前的状态：

```python
state_action = self.q_table.loc[observation, :]
```

然后就可以找这一行中的最大值对应的动作了。

但是这里有一个问题，如果有多个相同的最大值呢？这种情况下如果我们直接判断 argmax，返回的将永远是第一个最大值，永远不会遍历到后面的最大值对应的动作。所以这里我们需要随机选择一个最大值来避免这种情况：

```python
action = np.random.choice(state_action[state_action == state_action.max()].index)
```

这就是基于 Q 表选择的动作了。

在另一个分支进行随机选择：

```python
action = np.random.choice(self.actions)
```

最后返回执行的动作：

```python
return action
```

完整代码：

```python
def choose_action(self, observation):
    observation = str(observation)
    self.check_state_exist(observation)
    # action selection
    if np.random.uniform() < self.epsilon:
        # choose best action
        state_action = self.q_table.loc[observation, :]
        action = np.random.choice(state_action[state_action == state_action.max()].index)
    else:
        # choose random action
        action = np.random.choice(self.actions)
        return action
```

### `learn`

`learn` 是学习的过程，也就是更新 Q 表的函数，需要接收当前状态、当前执行的动作、获得的奖励和下一个状态 4 个参数：

```python
def learn(self, s, a, r, s_):
```

同样对状态做字符串化处理，并判断下一个状态是否在 Q 表中：

```python
s = str(s)
s_ = str(s_)
self.check_state_exist(s_)
```

根据上一部分的分析，更新 Q 表需要知道当前选择对应的最大奖励值（$r+\gamma max_{a'}Q(s',a')$，记作 `q_target`）和 Q 表预测的最大奖励值（$Q(s,a)$，记作 `q_predict`）。

其中 `q_predict` 比较容易得到：

```python
q_predict = self.q_table.loc[s, a]
```

而 `q_target` 需要判断下一状态是否到了最终状态，如果是，则 `q_target` 直接为当前获得的奖励 `r`，否则根据公式计算：

```python
if s_ != 'terminal':
    q_target = r + self.gamma * self.q_table.loc[s_, :].max()  # next state is not terminal
else:
    q_target = r  # next state is terminal
```

更新 Q 表：

```python
self.q_table.loc[s, a] += self.lr * (q_target - q_predict)
```

## 参考

[强化学习 (Reinforcement Learning) | 莫烦Python](https://mofanpy.com/tutorials/machine-learning/reinforcement-learning/)

## 鸣谢

[GitHub Copilot · Your AI pair programmer](https://copilot.github.com/)

![copilot-qlearning](https://i0.hdslb.com/bfs/album/149d09fab5e7742af1c45be2ef804110282b2bcb.jpg)
