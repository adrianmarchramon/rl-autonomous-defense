# Phase 4 — Improving the Agent: Reward Shaping and Tuning

> In this phase, you transform an agent that *works* into one that *defends well*. To do this, you will use the skill that most distinguishes an RL expert: **reward shaping** (reward design). It is crucial to understand why this is so important, as it is the core of this phase. In reinforcement learning, the reward *is* the task definition: the agent optimizes exactly what you reward, nothing more, nothing less. Therefore, designing the reward well is, in reality, defining *what "defending well" means* to the agent. A poorly designed reward causes the agent to learn something unintended (or nothing at all), while a well-conceived reward guides it toward your target strategy. It is subtle, represents your highest-impact lever, and mastering it is what separates those who simply apply RL from those who truly understand it. Alongside reward shaping, you will tune hyperparameters, compare algorithms, and iterate until you achieve a genuinely effective defender.

**Phase Objective:** Transform an agent that works into one that defends well.  
**Duration:** Weeks 5-6.  
**By the end, you will have:** A significantly more effective defending agent, resulting from careful reward design, hyperparameter tuning, and algorithm selection.

---

## The Big Picture

The workflow for this phase applies the improvement levers and measures their impact:

```
   Working agent (Phase 3)
        │
        ├──► [ 1-2. Reward shaping ]  ──►  designing the reward (the key lever)
        ├──► [ 3. Action reduction + hyperparameters ]  ──►  facilitating and tuning learning
        └──► [ 4. Compare algorithms ]  ──►  PPO vs. DQN
        │
        ▼
   [ 5. Iterate and measure ]  ──►  a significantly more effective defender
```

---

## Step 1 — Reward Shaping: Why Reward is Everything

Before making any changes, establish this core concept, as it is the most critical of this phase. An RL agent has no prior notion of what you consider "good defense": it only seeks to maximize cumulative reward. This means the reward is your *only* way to communicate what you want, and the agent will do *exactly* what the reward prioritizes, sometimes with inconvenient literalness. If the reward captures your actual objective well, the agent will learn to defend properly; if it is poorly defined, the agent will learn to "exploit points" in ways that are unhelpful.

There is a specific challenge in this environment that makes reward shaping necessary: **signal sparsity**. The strongest penalty (the attacker impacting `Op_Server0`) occurs late, at the end of a long sequence of actions, and infrequently. This makes learning difficult because the agent receives little feedback on whether its intermediate decisions are good (the **sparse rewards** problem). Reward shaping largely consists of adding intermediate signals that guide the agent along the way, without waiting for the final penalty.

---

## Step 2 — Designing a Better Reward

With that concept clear, you will design a reward that guides the agent more effectively. CybORG's base reward already penalizes host compromise; you will enrich it with nuances that better reflect your true objectives. You can do this using an environment wrapper:

```python
import gymnasium as gym


class ShapedRewardWrapper(gym.Wrapper):
    """Enriches the reward to better guide the defender."""

    def step(self, action):
        obs, reward, terminated, truncated, info = self.env.step(action)
        reward += self._shaping(obs, action, info)
        return obs, reward, terminated, truncated, info

    def _shaping(self, obs, action, info) -> float:
        extra = 0.0
        # Examples of intermediate signals (to be adjusted carefully):
        # - reward restoring/cleaning a genuinely compromised host (timely and useful action)
        # - slightly penalize restoring without need (Restore is disruptive)
        # - reward keeping Op_Server0 clean (the critical asset)
        ...
        return extra
```

The design of the signals is where domain judgment is required. The idea is to reward useful actions at the right time (such as restoring a compromised host before the attacker advances) and to reflect the **real costs** of actions (remember from Phase 2 that `Restore` is powerful but disruptive, so abusing it should incur a cost; also, `Remove` only works against user-level intruders, not root). A good reward pushes the agent toward the correct strategic trade-off: acting on time, with the appropriate action, without wasting resources.

However, there is a well-known RL pitfall to watch out for, and addressing it demonstrates maturity: **reward hacking**, where the agent "cheats" your reward function. If you design the signals poorly, the agent will find ways to maximize reward without actually defending. For example, if you over-reward the restore action, it might compulsively restore hosts continuously (earning your shaped reward, but at an absurd operational cost); if you over-penalize the cost of acting, it might learn to do nothing at all. This is why reward shaping is delicate: every signal you add can have unforeseen consequences, and you must observe the agent's behavior to detect whether it is optimizing the letter of the reward rather than its spirit. (There is a principled technique, *potential-based reward shaping*, which allows you to add guiding signals without altering the optimal policy; this is a great reference if you want to dig deeper.) The art lies in designing a reward that captures your true objective with enough fidelity that it cannot be bypassed.

---

## Step 3 — Other Levers: Reducing the Action Space and Tuning Hyperparameters

In addition to the reward, you have two other levers. The first, which you saw in Phase 2, is **reducing the action space**. The full space is massive (one action for every action-host combination), which makes learning extremely difficult. Restricting it to a relevant subset (focusing on analyzing and restoring enterprise servers and `Op_Server0`, which is what actually matters) makes the problem much more approachable, and this is exactly what teams addressing this challenge do. It is a way to simplify the problem so that the agent can learn more effectively.

The second lever is **hyperparameter tuning**, the settings that control how the algorithm learns. In PPO, the most important parameters are the learning rate (`learning_rate`), how many steps it collects before each update (`n_steps`), the batch size (`batch_size`), the discount factor (`gamma`), and the network size:

```python
model = PPO(
    "MlpPolicy", env,
    learning_rate=3e-4,    # step size of updates
    n_steps=2048,          # experience gathered before updating
    batch_size=64,
    gamma=0.99,            # weight of future rewards (higher = more far-sighted)
    tensorboard_log="./logs/",
)
```

Tuning these values (either systematically or by starting from pre-tuned values published in resources like Stable-Baselines3's RL Zoo) can significantly improve learning. A good starting point is to avoid changing everything at once; instead, modify one hyperparameter at a time and observe its effect on the training curves to understand what helps.

---

## Step 4 — Comparing Algorithms

An improvement that demonstrates rigor is comparing different algorithms. You trained with PPO, but it is not the only option: **DQN** (and its DDQN variant) is another deep RL algorithm from a different family that also performs well in this challenge. To explain the conceptual difference: PPO directly learns a policy and is robust and stable; DQN learns action values and can be more data-efficient, though sometimes less stable. Which one works better for this specific problem is an empirical question worth answering:

```python
from stable_baselines3 import DQN, PPO

# Train both under the same conditions and compare their performance as defenders
```

Training both under equal conditions and comparing their performance (which one learns a better defender, faster, and more stably) is a useful experiment that provides valuable insights and demonstrates that you are not simply applying a default algorithm, but rather choosing with clear criteria. It also makes for excellent README content.

---

## Step 5 — Iterating and Measuring Improvement

The work in this phase is iterative: you apply an improvement (a new reward shape, a tuning adjustment, a different algorithm), train, measure, and compare it against previous runs. The essential practice that brings rigor to this iteration is **always measuring**: evaluate each version of the agent consistently (using `evaluate_policy` over a sufficient number of episodes) and compare it to the previous one to determine if the change truly helps.

```python
from stable_baselines3.common.evaluation import evaluate_policy

mean, std = evaluate_policy(model, env, n_eval_episodes=50)
print(f"Mean reward: {mean:.1f} (±{std:.1f})")  # compare against the previous version
```

The goal is to move closer to a genuinely effective defender that approaches the benchmarks of the challenge (around -16 against `B_line`). It is important to maintain the same honesty as in Phase 2: "effective" means noticeably better, approaching those solid benchmarks, not perfect, as the defender cannot prevent all damage. Aiming for the impossible (a reward of zero) would mean failing to understand the problem; aiming to approach the best known benchmarks is the correct target.

---

## Verification: Definition of Done

This phase is complete when the following conditions are met:

- [ ] You have designed an improved reward function (reward shaping) that guides the agent more effectively, while keeping reward hacking in check.
- [ ] You have applied other levers: reducing the action space and tuning hyperparameters.
- [ ] You have compared at least two algorithms (PPO vs. DQN) under identical conditions.
- [ ] You have iterated, consistently measuring the performance improvement of each version.
- [ ] Your agent defends significantly better than the one from Phase 3, approaching established benchmarks.
- [ ] **The key test:** Your agent defends the network significantly more effectively as a result of careful reward design and tuning.

The key test is the qualitative leap: if your agent, after this phase, defends the network noticeably more effectively than the one that simply "worked," and you can attribute that improvement to specific decisions (a better reward, a hyperparameter adjustment, a different algorithm), you have demonstrated the skill that truly distinguishes an RL practitioner. It is not just about training an agent; it is about knowing how to make it perform well.

---

## Deliverables and Next Steps

Upon completing Phase 4, you will have a capable defender: an agent that is significantly more effective, resulting from careful reward design (the most distinctive skill in RL), hyperparameter tuning, action space reduction, and an informed choice of algorithm. You have learned how to go beyond simply training an agent to actively *improving* it, while navigating subtle pitfalls like reward hacking and rigorously measuring every step. You have a defender you can be confident in.

The next step, **Phase 5**, puts it to the ultimate test: **evaluation against adversaries**. Until now, you have trained and evaluated primarily against a single attacker. Now, you will rigorously and honestly measure how well your agent defends against *different* attackers (including those not seen during training), compare it against baselines, and verify whether its policy **generalizes** or if it only knows how to defend against what it already knows. This is the phase that distinguishes an agent that looks good from one that actually is, and where you will apply the rigorous evaluation standards cultivated in your previous projects. You have progressed from "having an effective defender" to being on the verge of "knowing exactly how good it is, against whom, and where its limits lie."
