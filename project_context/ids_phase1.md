# Phase 1 — RL Fundamentals with a Simple Environment

> In this phase, you learn real RL, hands-on. Reinforcement learning is abstract and difficult, and the best way to understand it is not to read about it, but to *build a complete RL system from scratch* on a problem so simple that you can see everything that happens: every state, every action, every reward, and every step of the learning process. That is why here you build your own toy defense environment and your own Q-learning agent, without libraries hiding the mechanics. The guiding idea of this phase is this: implementing the algorithm yourself, instead of calling a function, is what turns RL from a black box into something you understand from the inside out. It is an investment that pays off handsomely when you get to CybORG, because you will understand what deep RL is doing there. And, along the way, the toy problem will reinforce the idea of Phase 0: it is genuinely sequential (actions change the future), not a classification problem.

**Phase objective:** to understand real RL, hands-on, before moving on to the complex environment.
**Duration:** weeks 1-2.
**By the end, you will have:** a simple defense environment and a Q-learning agent implemented by you that learns to defend it, along with a solid, firsthand understanding of how an RL agent learns.

---

## The Big Picture

The flow of this phase builds the two components of RL and sets them to learn:

```
   [ 1. Simple environment ]  ──►  a toy defense problem (with Gymnasium)
        │                          healthy/compromised machines; the defender cleans them
        ▼
   [ 2. Q-learning agent ]   ──►  implemented by you (the Q-table, the learning rule)
        │
        ▼
   [ 3. Training loop ]      ──►  the agent interacts and learns, exploring less and less
        │
        ▼
   [ 4. Seeing it learn ]    ──►  the reward curve goes up; the policy makes sense
```

---

## Step 1 — The Simple Environment

You begin by building a toy defense environment using the Gymnasium API, simple enough to see everything but capturing the essence of defense. The problem: there are a few machines, each either healthy or compromised; at each step, the attacker can compromise a healthy machine, and the defender (the agent) can clean a machine to restore it, or do nothing. The goal is to keep the machines healthy without wasting actions. Create `src/envs/simple_defense.py`:

```python
import gymnasium as gym
import numpy as np
from gymnasium import spaces


class SimpleDefenseEnv(gym.Env):
    """Toy defense: N machines that can be compromised; the defender cleans them."""

    def __init__(self, n_machines: int = 3, attack_prob: float = 0.3, max_steps: int = 50):
        super().__init__()
        self.n_machines = n_machines
        self.attack_prob = attack_prob
        self.max_steps = max_steps
        # Observation: the state (healthy=0 / compromised=1) of each machine
        self.observation_space = spaces.MultiBinary(n_machines)
        # Actions: do nothing (0), or clean machine i (1..N)
        self.action_space = spaces.Discrete(n_machines + 1)

    def reset(self, seed=None, options=None):
        super().reset(seed=seed)
        self.machines = np.zeros(self.n_machines, dtype=np.int8)  # all healthy
        self.steps = 0
        return self.machines.copy(), {}

    def step(self, action: int):
        reward = 0.0
        # 1. Defender's action
        if action > 0:                          # clean machine (action - 1)
            self.machines[action - 1] = 0
            reward -= 0.1                       # actions have a small cost
        # 2. Attacker can compromise a healthy machine
        if self.np_random.random() < self.attack_prob:
            healthy = np.where(self.machines == 0)[0]
            if len(healthy) > 0:
                self.machines[self.np_random.choice(healthy)] = 1
        # 3. Reward: penalty for each compromised machine
        reward -= float(self.machines.sum())
        # 4. End of episode
        self.steps += 1
        terminated = self.steps >= self.max_steps
        return self.machines.copy(), reward, terminated, False, {}
```

Notice how this small environment embodies RL concepts and reinforces the Phase 0 approach. The **state** is which machines are compromised; the **actions** are cleaning a machine or doing nothing; the **reward** penalizes having compromised machines (and charges a small cost to act, so the agent doesn't waste actions). And, crucially, it is **sequential**: cleaning a machine now affects the future state, and the agent pursues a long-term goal (keeping the network healthy throughout the episode). It is not classification; it is the problem of deciding *what to do over time*, which is exactly where RL shines. 

Regarding the Gymnasium API, note the details of the current version: `reset` returns `(observation, info)` and `step` returns five values (`observation, reward, terminated, truncated, info`). A design note: this environment is fully observable (the agent sees which machines are compromised), which is a deliberate simplification for learning; the actual CybORG environment will add the challenge of partial observability (not directly seeing the intrusions).

---

## Step 2 — The Q-learning Agent

Now you will implement the agent, and here is the heart of this phase: you are going to write the learning algorithm yourself, rather than calling a library, because that is what will make you understand it. You will use tabular Q-learning, the simplest RL algorithm. The idea: the agent maintains a **Q-table** that estimates, for each state and action, how good that action is in that state (how much total future reward it can expect). Create `src/agents/q_learning.py`:

```python
from collections import defaultdict

import numpy as np


class QLearningAgent:
    def __init__(self, n_actions: int, alpha=0.1, gamma=0.95, epsilon=1.0):
        self.n_actions = n_actions
        self.alpha = alpha        # learning rate: how much to update each time
        self.gamma = gamma        # discount factor: how much the future matters
        self.epsilon = epsilon    # exploration probability (try something at random)
        self.Q = defaultdict(lambda: np.zeros(n_actions))

    def _key(self, obs):
        return tuple(int(x) for x in obs)   # state as table key

    def choose_action(self, obs) -> int:
        # epsilon-greedy: sometimes explores, sometimes exploits what it has learned
        if np.random.random() < self.epsilon:
            return np.random.randint(self.n_actions)        # explore
        return int(np.argmax(self.Q[self._key(obs)]))       # exploit

    def update(self, obs, action, reward, next_obs):
        s, s_next = self._key(obs), self._key(next_obs)
        best_future = np.max(self.Q[s_next])
        # The Q-learning rule: brings the estimate closer to what was actually observed
        target = reward + self.gamma * best_future
        self.Q[s][action] += self.alpha * (target - self.Q[s][action])
```

It is worth thoroughly understanding the **update rule**, because it is how the agent learns, and it is more intuitive than it looks. The value `Q[s][a]` estimates the total reward to expect after taking action `a` in state `s`. Every time the agent acts, it adjusts this estimate to bring it closer to what it actually observed: the immediate `reward` plus the best it can achieve from the next state (`gamma * best_future`). Repeating this thousands of times, the estimates converge to the actual values, and then acting by always choosing the action with the highest Q yields the optimal strategy. 

The **discount factor** `gamma` controls how much the future is weighted against the present (a high gamma makes the agent forward-looking). This simple rule, applied over and over, is all it takes for the agent to learn from rewards, even delayed ones.

And take a look at the `choose_action` method: it implements the **exploration vs. exploitation** dilemma that you saw in the fundamentals. With a probability of `epsilon`, the agent tries a random action (explores, to discover what works); the rest of the time, it uses what it has already learned (exploits). This is a direct and concrete encounter with one of the core concepts of RL.

---

## Step 3 — The Training Loop

With the environment and the agent ready, you tie them together in the training loop, which is the RL cycle in action: the agent acts, observes the result, updates its table, and repeats, over many episodes. Create `src/training/train.py`:

```python
import numpy as np


def train(env, agent, n_episodes=5000, epsilon_min=0.05, epsilon_decay=0.999):
    rewards = []
    for _ in range(n_episodes):
        obs, _ = env.reset()
        total, terminated = 0.0, False
        while not terminated:
            action = agent.choose_action(obs)
            next_obs, reward, terminated, _, _ = env.step(action)
            agent.update(obs, action, reward, next_obs)   # learns from each step
            obs = next_obs
            total += reward
        # Decay exploration over time: explore a lot at the beginning, little at the end
        agent.epsilon = max(epsilon_min, agent.epsilon * epsilon_decay)
        rewards.append(total)
    return rewards
```

There is an important detail that shows you understand RL: the **gradual decay of epsilon**. At the beginning of training, the agent knows nothing, so it should explore extensively (high epsilon) to discover what works. As it learns, it should explore less and exploit more of what it has learned (epsilon gradually decreases). This transition from exploring to exploiting is exactly how the central dilemma of RL is resolved in practice, and it is key to helping the agent converge to a good policy.

---

## Step 4 — Seeing It Learn

The most satisfying part: verifying that the agent actually learns. To do this, you look at two things. First, the **learning curve**: the total reward per episode should rise as the agent improves its strategy.

```python
import matplotlib.pyplot as plt

from src.agents.q_learning import QLearningAgent
from src.envs.simple_defense import SimpleDefenseEnv
from src.training.train import train

env = SimpleDefenseEnv()
agent = QLearningAgent(n_actions=env.action_space.n)
rewards = train(env, agent)

moving_average = np.convolve(rewards, np.ones(100) / 100, mode="valid")
plt.plot(moving_average)
plt.xlabel("Episode"); plt.ylabel("Average Reward (100 episodes)")
plt.title("The agent learns to defend")
```

Seeing that curve rise is visceral proof that learning works: starting from acting completely at random, the agent discovers an increasingly better strategy on its own. The second thing you look at is the **learned policy**: what the agent decides to do in each situation. You can inspect it and verify that it makes sense:

```python
# What does the agent do when only machine 0 is compromised?
state = (1, 0, 0)
action = int(np.argmax(agent.Q[state]))
print(f"Preferred action: {action}")   # we expect 1 = clean machine 0
```

Verifying that the agent, when faced with a compromised machine, has learned to clean it on its own (without anyone telling it to do so, relying solely on rewards) is the moment RL stops being abstract and becomes tangible. You have watched an agent *discover* a strategy. That is exactly what this phase wants you to understand.

---

## Step 5 — Tests

Verify that the environment and the agent behave as expected. Complete `tests/test_simple_rl.py`:

```python
import numpy as np

from src.agents.q_learning import QLearningAgent
from src.envs.simple_defense import SimpleDefenseEnv


def test_env_starts_healthy_and_cleans():
    env = SimpleDefenseEnv(n_machines=3, attack_prob=0.0)  # no attacks, to isolate
    obs, _ = env.reset()
    assert obs.sum() == 0                      # all machines healthy at start
    env.machines[1] = 1                        # compromise machine 1
    obs, _, _, _, _ = env.step(2)              # action 2 = clean machine 1
    assert obs[1] == 0                         # ended up clean


def test_q_update_learns_from_reward():
    agent = QLearningAgent(n_actions=4, epsilon=0.0)
    obs, next_obs = (1, 0, 0), (0, 0, 0)
    q_before = agent.Q[(1, 0, 0)][2]
    agent.update(obs, action=2, reward=5.0, next_obs=next_obs)  # good reward
    assert agent.Q[(1, 0, 0)][2] > q_before     # estimate increases after a positive reward
```

The first test checks that the environment works (starts healthy, and cleaning restores a machine); the second, that the Q-learning rule actually learns (after a positive reward, the estimate for that action increases). These are verifications that both components of RL work. Run them with `make test`.

---

## Verification: The "Done" Criteria

The phase is complete when the following is met:

- [ ] You have built a simple defense environment with the Gymnasium API.
- [ ] You have implemented a Q-learning agent from scratch (the Q-table, the update rule, epsilon-greedy).
- [ ] The training loop ties both together and decays exploration over time.
- [ ] You have seen the agent learn (the reward curve goes up) and have inspected its policy.
- [ ] **The key test:** you understand and can explain how an RL agent learns, because you built one from scratch.

The key test is not whether the agent works, but whether *you understand* how it learns. If you can explain the loop (observe, act, reward, update), the Q-learning rule (bringing the estimate closer to what was observed), and the exploration-exploitation dilemma (and you have implemented them, not just read about them), you have achieved what this phase aims for: turning RL from an abstract concept into something you understand inside out. This understanding is the foundation upon which all work with the realistic environment will build.

---

## Deliverables and What's Next

By closing Phase 1, you have something more valuable than a toy agent: you have a solid, firsthand understanding of RL, earned by building a complete system from scratch. You have seen an agent discover a defense strategy solely from rewards, you have implemented the learning mechanics yourself, and you have made tangible the concepts (the loop, the policy, exploration) that were previously abstract. This foundation is what will make the complex environment approachable.

The next step, **Phase 2**, takes you to the realistic environment: **the CybORG / CAGE defense environment**. You will leave the toy problem behind and face a simulated enterprise network, where a sophisticated red agent attempts to compromise it and your blue agent must defend it, with much richer observation and action spaces, and with partial observability. You won't train yet (that is Phase 3): first you will understand the environment, just as you did with your simple environment, but now with the depth that comes from RL no longer being foreign to you. You have gone from "I understand how an RL agent learns" to being on the verge of "I understand the realistic environment where my agent will learn to actually defend."
