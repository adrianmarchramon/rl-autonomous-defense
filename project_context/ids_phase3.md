# Phase 3 — Training the Defender Agent

> In this phase, the agent comes to life: you set it to learn how to defend the network for real. It is useful to keep in mind the relationship with what you have already done, as there are two changes compared to Phase 1, both for good reasons. The first: the RL loop is exactly the same one you understood with your toy environment (observe, act, reward, learn), but now the agent's "brain" is no longer a table, but a **neural network**, because the environment is too complex for a table. That is Deep RL. The second: in Phase 1, you wrote the algorithm yourself to understand it; now, for Deep RL, you will use a library of robust implementations (Stable-Baselines3), because these algorithms are complex and easy to get wrong, and the sensible approach is to rely on tested code and focus on the problem. The guiding idea of this phase is that combination: you understand the loop from the inside (Phase 1), and now you apply a robust tool to solve the difficult problem, watching your agent learn.

**Phase goal:** to train an agent that learns how to defend.
**Duration:** weeks 3-5.
**By the end, you will have:** a defender agent trained with Deep RL (using Stable-Baselines3 on CybORG), which has learned a defense policy and performs clearly better than at the start.

---

## The Big Picture

The workflow of this phase gets the agent learning and monitors its progress:

```
   [ 1. From Table to Networks ]  ──►  Deep RL: a neural network as the brain
                │
                ▼
   [ 2. Connect SB3 to the Env ]  ──►  Monitor + check_env + PPO on CybORG
                │
                ▼
   [ 3. Train                  ]  ──►  model.learn(...) against an attacker, with TensorBoard
                │
                ▼
   [ 4. Monitor the Curves     ]  ──►  average reward increases (even if negative)
                │
                ▼
   [ 5. Save and Verify        ]  ──►  agent clearly beats random
```

Add the Deep RL dependencies:

```bash
uv add stable-baselines3 tensorboard
```

---

## Step 1 — From Table to Networks: Why Deep RL

Before writing code, it is important to understand why the approach changes compared to Phase 1, as this is an important conceptual point. In your toy environment, the state space was tiny (a few machines, a few possible states), so a Q-**table** storing a value for every state and action worked perfectly. In CybORG, however, the state space is huge (an entire network, with many hosts in many possible states) and also partially observable: a table would be impossible, it wouldn't fit, and you would never see the exact same state twice. The solution is **Deep RL**: instead of a table, a **neural network** that *approximates* the function (what to do in each situation), generalizing to states it has not exactly seen before. The learning loop is the same one you already know; what changes is that the learning brain is a neural network capable of handling this complexity. Understanding this—that Deep RL is the same RL, but with a neural network to scale to large problems—is exactly what your work in Phase 1 allows you to see clearly.

---

## Step 2 — Connect Stable-Baselines3 to the Environment

Because you wrapped CybORG to make it compatible with Gymnasium in Phase 2, connecting Stable-Baselines3 to it is straightforward, since SB3 works with the standard Gymnasium interface. There are two best practices you should apply before training. First, wrap the environment with **Monitor**, which automatically logs the rewards and episode lengths (which will later generate your learning curves). Second, pass the environment through **check_env**, an SB3 utility that validates whether it meets the interface SB3 expects and warns you about potential issues: catching a compatibility error here, before starting training, saves you hours of debugging. Create `src/training/train.py`:

```python
from stable_baselines3 import PPO
from stable_baselines3.common.env_checker import check_env
from stable_baselines3.common.monitor import Monitor

from src.envs.cyborg_wrapper import make_cyborg_env

# The CybORG environment, wrapped for Gymnasium (Phase 2). We train against an attacker.
env = make_cyborg_env(red_agent="B_line")   # we start with B_line
env = Monitor(env)                            # logs rewards and episode lengths

check_env(env)   # validates that the environment complies with the interface expected by SB3
```

A design note: you are training against **one** specific attacker (start with B_line, the direct one). Confronting the agent with multiple attackers and checking whether it generalizes is the goal of Phase 5; here, the important thing is to get it to learn how to defend against a single adversary.

---

## Step 3 — Train

With the environment connected, you instantiate the algorithm and launch the training. You will use **PPO**, a robust and widely used Deep RL algorithm (and one of the best performing in this challenge); later, in Phase 4, you can compare it with alternatives like DQN. Since the observation is a vector (thanks to the wrapper from Phase 2), you use the `MlpPolicy` policy (a standard neural network):

```python
model = PPO(
    "MlpPolicy",                 # neural network for vector observations
    env,
    verbose=1,
    tensorboard_log="./logs/",   # to view the learning curves
)

model.learn(total_timesteps=500_000)         # Deep RL requires many interactions
model.save("models/defender_ppo_bline")
```

Two things that demonstrate your understanding of Deep RL: First, the number of **training steps** (`total_timesteps`) is large, because Deep RL needs a massive number of interactions with the environment to learn (unlike supervised learning, here the agent learns through trial and error, which requires a lot of experience). Training, therefore, takes some time. Second, you enable **TensorBoard** logging (`tensorboard_log`), which is what will allow you to monitor the learning in progress, as covered in the next step.

---

## Step 4 — Monitor the Learning: The Curves

Monitoring how the agent learns is not optional in RL: it is how you know if things are going well, if it has plateaued, or if something is broken. Run TensorBoard (`tensorboard --logdir ./logs/`) and observe the curves. There are three that you should know how to read, which is an RL skill in itself. The most important is **`ep_rew_mean`** (mean episode reward): this is the primary signal that the agent is learning, and it should **increase** during training. There is a crucial nuance here that ties back to what you learned in Phase 2: since the defender cannot prevent all damage, the rewards are **negative**, so "increasing" means becoming *less negative* (approaching, for example, that benchmark threshold of -16 against B_line). Do not be alarmed by negative numbers: what matters is the upward trend. The other two useful curves are **`ep_len_mean`** (mean episode length) and, specific to PPO, **`approx_kl`** (which monitors training stability: if it spikes, learning can become unstable).

Seeing the reward curve rise is proof that your agent is discovering, through trial and error, an increasingly better defense strategy, just like you saw in your toy environment, but now solving a real-world problem. If the curve does not rise (remains flat or drops), it is a sign that something needs to be reviewed (the reward, the hyperparameters, the action space), which is exactly the focus of Phase 4.

---

## Step 5 — Save and Verify the Agent

When training finishes, you save the agent (already handled by `model.save`) and verify that it has indeed learned by measuring its performance and comparing it against a baseline. SB3 comes with a convenient utility for this, `evaluate_policy`:

```python
from stable_baselines3.common.evaluation import evaluate_policy

mean_reward, std_dev = evaluate_policy(model, env, n_eval_episodes=20)
print(f"Mean reward of the trained agent: {mean_reward:.1f} (±{std_dev:.1f})")
# Compare it to a random or passive defender (from Phase 2 exploration): it will perform much worse.
```

The key verification is that your trained agent performs **clearly better** than the random or passive defender you observed in Phase 2 (whose penalties accumulated endlessly). This difference is proof that the agent has learned a useful defense policy: it is no longer acting randomly, but rather taking decisions that limit the attacker's damage. It does not have to be an excellent defender yet (you will fine-tune that in Phase 4), but it should have shown clear signs of learning.

---

## Verification: The "Definition of Done"

This phase is complete when the following are met:

- [ ] You have connected Stable-Baselines3 to the CybORG environment (incorporating Monitor and validating with check_env).
- [ ] You have trained an agent using Deep RL (PPO) against an attacker, over a large number of steps.
- [ ] You have monitored the learning process using TensorBoard and know how to read the curves (especially `ep_rew_mean`).
- [ ] The reward curve rises (becomes less negative): the agent is learning.
- [ ] The trained agent performs clearly better than a random or passive defender.
- [ ] **The key proof:** you have an agent that has learned a defense policy and performs better than when it started.

The key proof is demonstrated learning: if your agent, after training, defends the network clearly better than at the beginning (and much better than acting randomly), you have achieved what this phase set out to do, which is to bring the agent to life. The same RL loop you understood in your toy environment has now solved, with a neural network and a robust tool, a real defense problem. You now have an agent that has learned to defend.

---

## Deliverables and Next Steps

Upon wrapping up Phase 3, you have the heartbeat of the project running: a defender agent trained with Deep RL, which has learned a defense policy that limits the attacker's damage purely through interacting with the environment. You have bridged the gap from the toy environment to a realistic one, from tabular Q-learning to Deep RL with Stable-Baselines3, and you have seen (on the curves) your agent learn to defend a network. You have a working defender, ready to be refined.

The next step, **Phase 4**, takes it from "working" to "defending well": **improving the agent with reward shaping and tuning**. This involves one of the most distinctive skills in RL: **reward design**. As you saw, in RL a poorly designed reward ruins learning, while a well-designed one accelerates it; you will adjust the reward to guide the agent better, fine-tune the hyperparameters, and compare algorithms (PPO vs. DQN). You have gone from "I have an agent that is learning to defend" to being close to "I have an agent that defends really well, thanks to careful design."
