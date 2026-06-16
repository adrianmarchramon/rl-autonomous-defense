# Phase 2 — The Defense Environment (CybORG / CAGE)

> In this phase, you leave the toy problem behind and face a realistic environment, but with a key difference compared to what you will do next: **here, you seek to understand; you do not train yet**. Just as in Phase 1 you spent time understanding your simple environment before letting the agent learn, now you will do the same with CybORG, which is far richer: a simulated corporate network where a sophisticated attacker tries to make their way to a critical server, and your defender must stop them. The guiding idea of this phase is that you cannot design a good agent (nor a good reward, nor a good evaluation) without deeply understanding what it observes, what it can do, and what it seeks. And there is a leap in realism that defines this environment: unlike your toy problem, where the defender could see the actual state of each machine, here the observation is **partial**: the defender does not directly see what is compromised, but rather clues, and must act under uncertainty. Deeply understanding this is the goal of this phase.

**Phase Objective:** Set up the realistic network environment.
**Duration:** Weeks 2-3.
**By the end, you will have:** CybORG (CAGE Challenge 2) installed and running, along with a clear understanding of the network, what the defender observes and can do, the reward structure, and the scenario (what the attacker does and what the defender aims for).

---

## The Big Picture

The flow of this phase involves installing and unpacking the realistic environment:

```
   [ 1. Install CybORG / CAGE-2 ]  ──►  + wrappers to speak Gymnasium
        │
        ▼
   [ 2. The network and scenario ]  ──►  red seeks Op_Server0; blue defends it
        │
        ├──► [ 3. What the defender sees ]  ──►  PARTIAL observation (clues, not the truth)
        └──► [ 4. What it can do + reward ]  ──►  Monitor, Analyse, Remove, Restore...
        │
        ▼
   [ 5. Explore the running environment ]  ──►  see the red advance and the penalties
```

---

## Step 1 — Install CybORG / CAGE-2

You begin by installing the environment. CybORG (Cyber Operations Research Gym) is the simulation engine, and CAGE Challenge 2 is the scenario you will use. A tip that demonstrates your understanding of the field: it is best to use **CybORG++** (the bug-fixed version of CAGE 2 maintained by the Alan Turing Institute), as the original version had a few known bugs (for example, the action to remove an intruder was always reported as successful, whether it actually was or not). Install it following the instructions in its repository:

```bash
git clone https://github.com/alan-turing-institute/CybORG_plus_plus
# install according to repository instructions (via pip)
```

An important piece that connects to what you already know: CybORG does not directly expose the Gymnasium interface you learned in Phase 1; instead, it provides **wrappers** that adapt it. One wrapper structures the observation (presenting it, for instance, as a table per host), another adapts the action space, and another makes the entire package compatible with the standard Gymnasium API. Thanks to these, you can interact with CybORG using the same `reset` / `step` methods you already know, and connect it to standard RL algorithms. It is worth checking the examples in the repository to see how these wrappers are set up, as they are the bridge between the environment and the algorithms.

---

## Step 2 — The Network, the Scenario, and the Attackers

The most important part of this phase is understanding the scenario, so make sure to dedicate enough time to it. The CAGE Challenge 2 models a **small corporate network** divided into subnets: one containing user machines, another with company servers, and a critical operational server, **Op_Server0**, which is the primary asset to protect. The scenario is a two-player game:

The **red agent** (attacker) starts with an initial foothold: access to a user machine. From there, its goal is to make its way through the network (performing reconnaissance on other hosts, exploiting them, escalating privileges, and moving laterally) until it reaches and compromises ("impacts") Op_Server0. This is a step-by-step intrusion process. There are three attackers you should get to know, as you will be defending against them: **B_line**, which acts with prior knowledge of the network and heads directly and efficiently toward the target (the toughest attacker); **Meander**, which explores in a more methodical and erratic way (slower, broader); and **Sleep**, which does nothing (a baseline).

The **blue agent** (defender, yours) must prevent this: detect the intrusion and contain it, keeping the network (and especially Op_Server0) safe. Note that this is a genuinely sequential problem, which is exactly what RL leverages: the attacker progresses over time, and the defender must respond to a changing situation rather than classifying an isolated data point. This scenario reinforces, concretely and vividly, the approach you established in Phase 0.

---

## Step 3 — What the Defender Sees: Partial Observation

Here is the massive leap in realism compared to your toy environment, and it is crucial to understand it well because it dictates the entire difficulty of the problem. In your toy environment, the defender could see the actual state of each machine (healthy or compromised). In CybORG, **it does not**: the defender does not directly observe which hosts are compromised, but rather **clues**. Its monitoring action provides hints of suspicious network activity at each step, but not the clean truth. This is what is known as **partial observability**: the agent must *infer* the true state (Is there an intruder? Where? With what privilege level?) from noisy, incomplete evidence, and make decisions under that uncertainty.

This is a much greater challenge than the toy problem, and it is what makes the problem realistic (and difficult): in real-world defense, you never know with absolute certainty what is happening; you only have alerts and clues. CybORG's wrappers present this observation in a structured format (for instance, a table detailing what activity has been detected and what level of compromise is suspected for each host), which is what the agent receives as input. Understanding what information the defender has (and does not have) is essential to avoid expecting more from it than its sensors allow.

---

## Step 4 — What the Defender Can Do, and the Reward

The defender has a repertoire of **actions** that you should get to know, as they involve interesting strategic trade-offs that the agent will have to learn. The main actions are: **Monitor** (passively monitor to collect clues), **Analyse** (investigate a specific host in depth, providing much better evidence of an intrusion than simple monitoring), **Remove** (eliminate an intruder from a host), and **Restore** (restore a host from a backup). There is a nuance that demonstrates the depth of the environment: **Remove only works against intruders with user privileges, not root**, whereas **Restore is much more powerful (equivalent to starting fresh) but is disruptive and takes more steps**. This creates a genuine strategic dilemma: when faced with a compromised host, is a Remove action enough (fast, but potentially insufficient if the intruder is already root), or is a Restore needed (secure, but costly)? Learning to make this decision correctly is part of what will make the agent valuable.

A practical tip that shows foresight: the full action space is large (there is an action for every combination of action type and host), which makes learning difficult. A common technique used by teams tackling this challenge is to **reduce the action space** to a relevant subset (focusing on analyzing and restoring the corporate servers and Op_Server0, which is what truly matters) to make the problem more manageable. Keep this in mind for the training phase.

Regarding the **reward**, it reflects the damage: the defender receives penalties when the attacker compromises hosts and, most significantly, when it impacts Op_Server0. It is good to internalize a realistic expectation: the defender usually **cannot prevent all damage**, only minimize it; the best scores in this challenge are negative (around -16 against B_line and worse against Meander). Therefore, the goal is not a perfect defense, but rather limiting how far the attacker advances and how long they remain. Understanding this will save you from frustration when chasing the impossible.

---

## Step 5 — Explore the Running Environment

With the environment installed and understood on paper, run it to see it in action, just as you explored your simple environment. Execute a few steps—for example, with a defender that does nothing or acts randomly—simply to observe the dynamics:

```python
# Illustrative pattern: the environment wrapped to be compatible with Gymnasium
obs, info = env.reset()
for step in range(15):
    action = env.action_space.sample()      # random action, just to explore
    obs, reward, terminated, truncated, info = env.step(action)
    print(f"step {step}: reward = {reward}")
    if terminated or truncated:
        break
```

Observe how, with a passive or random defender, penalties accumulate as the attacker advances through the network toward Op_Server0. Also examine the observation (what clues the defender receives) and the action space (how many and which ones they are). This exploration makes everything you understood on paper tangible, giving you a baseline to compare against: this is what happens when defense *is not* handled well, and it is exactly what your agent will learn to improve in the coming phases.

---

## Verification: The "Definition of Done"

The phase is complete when the following are met:

- [ ] CybORG (CAGE Challenge 2, preferably CybORG++) is installed and working, wrapped to be compatible with Gymnasium.
- [ ] You understand the network and the scenario: the attacker aims for Op_Server0, while the defender protects it.
- [ ] You are familiar with the attackers (B_line, Meander, Sleep).
- [ ] You understand the **partial** observation (the defender sees clues, not the actual truth) and why that makes the problem difficult.
- [ ] You understand the defender's actions (including the Remove/Restore dilemma) and the reward (minimizing damage, not preventing it entirely).
- [ ] **The key test:** You understand what the defending agent observes, what it can do, and what it seeks in the realistic environment.

The key test lies in having a deep understanding of the environment: if you can explain what the defender sees (and what it does not see, due to partial observability), what actions it has and what trade-offs they imply, and what goal it pursues, you have the essential foundation needed to train it effectively. Just as with your simple environment, you cannot make an agent learn in an environment you do not understand; unpacking it thoroughly is what gives meaning to the training phase.

---

## Deliverables and What's Next

By closing Phase 2, you have the realistic environment up and running, and more importantly, deeply understood: you know what the network looks like, what the attacker is searching for, what the defender observes (partially), what actions it can take, and how it is rewarded. You have made the leap from your toy problem to a realistic defense scenario, and you have grasped the challenge that defines it (partial observability, action trade-offs, and the impossibility of a perfect defense). You have everything needed to start training your agent.

The next step, **Phase 3**, is where the agent comes to life: **training the defending agent**. You will connect Stable-Baselines3 (using a deep RL algorithm like PPO or DQN) to the CybORG environment, and you will train your blue agent while monitoring how it learns through its reward curves. The RL loop you understood in your simple environment is the same, but now the problem is complex enough to require neural networks instead of a lookup table. You have gone from "understanding the realistic environment" to being on the verge of "having an agent that learns to defend a real network."
