# Intrusion Detection with RL — Foundations and Design

> This document represents the conceptual framework prior to writing code. It covers what reinforcement learning and autonomous cyber defense actually are; the honest approach defining the project (using RL for sequential response, not classification); its research nature and the gap between simulation and reality; exactly what the system does and how; the use case and target audience; the technology stack justified component by component; and the repository architecture. Solidly understanding these foundations is what separates a project that simply "applies RL to a security dataset" from one that demonstrates a genuine grasp of what RL is, what it is useful for, and how it is applied thoughtfully to a cutting-edge problem.

> **The defining approach of this project, from the start:** applying RL to mere intrusion *classification* is widely criticized because it is essentially supervised learning in disguise, lacking the sequential and interactive structure that makes RL valuable. This project uses RL for its true purpose: the sequential problem of *response*—how to **defend** a network against an evolving intrusion. The agent learns a defense policy, not a label.

> **The nature of the project:** everything takes place in a **simulated** environment, avoiding the risks associated with real-world networks. It is important to be honest about the fact that a policy learned in simulation does not transfer directly to a real network (the simulation-to-reality gap is an open problem). Thus, this is a research demonstration, not a deployable defender. Additionally, because RL in cybersecurity is dual-use, the project deliberately focuses on defense.

---

## Introduction: What This Project Actually Is

To understand this project, we must begin with two concepts: reinforcement learning and autonomous cyber defense.

**Reinforcement learning** (RL) is a branch of AI distinct from those used in previous projects, and it is crucial to understand it well as it is the novel component here. The best way to grasp it is by analogy: RL is learning how to play a game *by playing it*, through trial and error, rather than being taught the correct moves. An **agent** interacts with an **environment**: it observes a **state** (the current situation), chooses an **action**, receives a **reward** (a signal indicating how well or poorly it performed) and a new state, and starts the cycle again. By repeating this loop thousands of times, the agent learns a **policy**: a way of choosing actions based on the state that maximizes cumulative reward over time.

The difference with supervised learning is the key to the entire project, so it is worth being precise. In supervised learning, you have labeled examples (an input and its correct output) and you learn to reproduce them. In RL, there is no "correct answer" to copy; there are only rewards, often delayed, indicating performance quality, and the agent must *discover* optimal behavior on its own through interaction. Crucially, **its actions affect what happens next**: the environment reacts, meaning the agent does not learn an isolated decision, but rather a *long-term strategy*. This sequential and interactive nature, where current decisions alter future situations, makes RL the appropriate tool for problems focused on *making sequential decisions within a responsive environment*. There is also a defining dilemma in RL, **exploration versus exploitation**: the agent must balance trying new actions (to discover better strategies) with leveraging what it already knows works.

**Autonomous Cyber Defense** (ACD) is precisely this type of problem, which explains the connection. It frames security as a game between two agents: a **red agent** (the attacker, who tries to compromise the network, gain access, move laterally between machines, and reach critical assets) and a **blue agent** (the defender, who tries to keep the network secure by monitoring, detecting intrusions, removing malware, restoring systems, and containing the attacker). The blue agent is the RL agent: it observes the state of the network, decides on defensive actions, sees how the attacker reacts, and learns a defense policy. It is a genuinely sequential problem because the attacker adapts, and the defender must adjust step by step.

### The Most Important Concept: Defending Is Not Classifying

There is a core concept that serves as the soul of this project, equivalent to the central tenets of prior projects: **RL is not meant for classifying intrusions; it is meant for learning how to defend against them**. Understanding this distinction is key to demonstrating conceptual maturity.

Many apply RL to intrusion detection in a naive way: they take a dataset of network flows, treat each flow as a "state", "classify as attack or normal" as the "action", and the reward as success or failure. The issue with this setup is that **the agent's action does not affect what happens next** (the subsequent flow is independent of the previous one). There is no sequence, no strategy, and no long-term goal; it is essentially a classifier with an unnecessarily convoluted training procedure. The entire interactive and sequential structure that makes RL valuable is missing, which is why that approach is simply supervised learning in disguise.

The *defense* approach, on the other hand, possesses this exact structure. The defender's actions (isolating a machine, terminating a process, restoring a system) **do alter the course of events**; the attacker **does respond**; and there is a sequence of decisions with a long-term goal (keeping the network secure). Therefore, RL is suited for response rather than classification, and this is why the project is framed around "detect and respond" rather than "classify". The agent does not learn to *recognize* attacks; it learns to *defend* against them. This distinction is the intellectual core of the project.

### Target Audience and Why It Resonates

As with other projects, it is useful to be explicit about the target audience. An **AI or machine learning** profile will value proficiency in RL (which is notably more challenging and less common than supervised learning), and especially the understanding of when to apply it (the correct paradigm, rather than the easy shortcut). A **security** profile will appreciate your work in autonomous cyber defense, a cutting-edge field with an inherent adversarial mindset. A **recruiter** will grasp the memorable narrative ("I trained an agent to defend a network"), the rarity of the skill set, and the focus on explainability. Lastly, everyone will appreciate the **maturity** of an honest approach that uses the right tool for the job and recognizes the project's actual scope.

With this in mind, every decision in this document points toward a single conclusion: for observers of the project to recognize that you understand reinforcement learning, apply it to a cutting-edge security problem, and truly comprehend what you are building.

---

## 1. What Exactly You Are Going to Build

The system is a defending agent that learns via reinforcement learning, structured as follows:

```
       Red Agent (Attacker) ⚔️   vs   🛡️ Blue Agent (Defender, RL)
            │                                 │
            ▼                                 ▼
       ┌──────────────────────────────────────────────────────┐
       │   Simulated Network Environment (CybORG / CAGE)      │
       │   the attacker attempts to compromise & move         │
       └──────────────────────────────────────────────────────┘
            │  observation: network state       ▲  reward: did it defend well?
            ▼                                   │
       [ RL Agent ]  ──►  observes, detects, and ACTS (monitor, isolate, restore...)
            │
            ▼
       [ Learned Defense Policy ]  ──►  evaluated, explained, and visualized
```

The core is the familiar **RL loop**, here applied to defense. The **environment** is the simulated network where the attacker operates. The **agent** observes the state of this network (which machines appear compromised, what suspicious activity is occurring), selects a defensive **action**, and receives a **reward** reflecting how well it is protecting the network. By repeating this loop across numerous simulated episodes, the agent learns a **defense policy**: a strategy for what action to take in each situation. Once learned, this policy is **evaluated** (does it defend effectively? against which attackers?), **explained** (what strategy did it discover?), and **visualized** (watching it defend step-by-step).

In essence, you are building a *defender that learns*, rather than a detector. The entire project revolves around helping it learn effectively, understanding what it learned, and presenting it.

---

## 2. The Use Case: Defending a Network with a Learning Agent

It is worth examining what makes this project compelling, as it combines several uncommon strengths.

### A Sophisticated and Uncommon AI Paradigm

Reinforcement learning is structurally more challenging and less common in portfolios than supervised learning or LLMs. It features its own theory and unique hurdles (such as exploration, reward shaping, and training stability), and developing a strong grasp of it is a sign of technical depth. Demonstrating its application to a complex, real-world scenario highlights a high level of competence. For your portfolio, this project incorporates a major branch of AI that was not previously represented.

### The Frontier of Research: Autonomous Defense

Autonomous cyber defense is an active research area supported by defense organizations, structured around reference environments (such as CybORG) and public benchmarks (the CAGE Challenges). Working in this domain shows that you are operating at the cutting edge, rather than on textbook problems, making it a topic that generates immediate interest.

### The Maturity of Choosing the Right Tool

As noted, many apply RL poorly to intrusion detection, turning it into disguised classification. Choosing the correct approach (sequential response) and explaining the reasoning behind it demonstrates something more valuable than just running an algorithm: it shows that you understand *the purpose of each tool* and do not apply patterns blindly. This conceptual maturity is precisely what distinguishes a strong AI engineer.

### A Memorable and Presentable Narrative

"I trained an AI agent to autonomously defend a network against attacks" is a highly memorable concept. Unlike many abstract AI projects, this one can be *visualized*: watching the agent defend step-by-step and *explaining* the strategy it discovered independently is engaging and makes a lasting impression.

### The Research Nature: Honesty Regarding Scope

This brings us to the responsible aspect of the project, which serves as both a caution and a mark of maturity. Everything takes place in simulation, avoiding the risks of interacting with real-world networks. However, we must remain honest about two points. First, the **sim-to-real gap**: a policy trained in a simulator does not transfer seamlessly to a real network, which is more complex, noisy, and unpredictable than any simulation. Therefore, this project is a proof of *approach*, not a production-ready defender, and stating this clearly shows you understand the true scope of your work. Second, the **dual-use nature**: RL in cybersecurity can also be used for offense (as red agents learn to compromise networks), which is why this project deliberately and explicitly focuses on **defense**. Acknowledging these limitations, rather than overstating success, distinguishes a rigorous project.

Additionally, this project connects with prior ones: it shares the **adversarial and cybersecurity mindset** of the honeypot project (the attacker-defender game) and builds upon the concept of **explainability** from the medical project (making an otherwise opaque policy understandable). It is the piece that integrates RL into your portfolio.

---

## 3. The Technology Stack, Justified Component by Component

As in previous projects, choosing tools carefully and understanding the rationale behind them demonstrates solid technical judgment. The **philosophy** organizing this stack is twofold: utilizing standard RL tools (the environment interface, algorithm implementations) and the reference environment for autonomous defense, but starting with simple setups to ensure a thorough understanding before introducing complexity.

### Gymnasium: The Environment Interface

Gymnasium (the successor to OpenAI Gym) is the standard API for RL environments. It defines the common contract for states, actions, and rewards—the "language" that allows any RL algorithm to communicate with any environment. It serves as the foundation for everything, and the fact that CybORG follows this API allows you to train standard agents on it without friction.

### CybORG and the CAGE Challenges: The Defense Environment

CybORG (Cyber Operations Research Gym) is the reference simulator for autonomous cyber defense. It models an enterprise network where a red agent attempts to compromise machines and move laterally, and a blue agent must detect and respond. The CAGE Challenges are a series of public challenges of increasing complexity using CybORG. A key design decision: for this project, the **CAGE Challenge 2** (one defender against one attacker in an enterprise network) is the ideal entry point because it is the most widely studied and has an approachable level of difficulty; CAGE 4, by contrast, is multi-agent and far more complex, making it better suited as a future objective.

### Starting Simple: A Custom Environment and Q-learning

This represents a practical pedagogical decision that is important to grasp. RL can be challenging, and jumping straight into CybORG would be like learning to swim in open ocean. Therefore, you will start by building a **simple custom environment** (a toy defense problem) using Gymnasium and solving it with the simplest algorithm available, **tabular Q-learning**, implemented from scratch. The goal is not the toy environment itself, but rather *truly understanding RL fundamentals* (the loop, rewards, policy, exploration) in a scenario where you can trace every event, before confronting realistic environments and complex algorithms. This is the difference between understanding RL deeply versus simply running a library as a black box.

### Stable-Baselines3: Deep RL

Once transitioning to CybORG, the problem becomes too complex for tabular Q-learning, requiring *deep* RL (utilizing neural networks). For this, you will use Stable-Baselines3, the standard library offering solid, thoroughly tested implementations of deep RL algorithms like PPO and DQN. Reimplementing these algorithms would be risky and prone to subtle bugs; utilizing reference implementations allows you to focus on what truly matters in the project: the environment, reward design, and evaluation.

### Visualization

To understand and showcase the behavior, you will use visualization tools: matplotlib for learning curves (tracking how the agent improves during training) and Streamlit (which you already know) to visualize the agent defending step-by-step.

### Stack Summary

| Layer | Tool | What it solves |
|------|-------------|--------------|
| Environment Interface | Gymnasium | The standard state-action-reward contract |
| Defense Environment | CybORG (CAGE 2) | The simulated network to defend |
| Simple Environment | Gymnasium (custom) | Learning RL fundamentals |
| Algorithms | Stable-Baselines3 (PPO, DQN) | Deep RL, solid and proven |
| Baseline Algorithm | Tabular Q-learning | Understanding the foundations |
| Visualization | matplotlib, Streamlit | Curves and agent visualization |
| Quality | uv, pytest, ruff, Docker | Reused from your previous projects |

---

## 4. Repository Architecture

As with previous projects, a clean structure communicates professionalism and is organized around the project workflow:

```
rl-cyber-defense/
├── src/
│   ├── envs/
│   │   ├── simple_defense.py   # simple custom environment (for learning)
│   │   └── cyborg_wrapper.py   # adapter for the CybORG environment
│   ├── agents/
│   │   ├── q_learning.py       # tabular Q-learning agent (fundamentals)
│   │   └── deep_agents.py      # deep RL agents (Stable-Baselines3)
│   ├── training/train.py       # the training loop
│   ├── evaluation/evaluate.py  # evaluation against adversaries
│   ├── explain/policy.py       # explain and visualize the policy
│   └── config.py
├── dashboard/app.py            # visualization of the defending agent
├── models/                     # trained agents (gitignored)
├── docs/
│   ├── context.md              # autonomous defense and the RL approach
│   └── scope_and_limits.md     # research nature and the sim-to-real gap
├── notebooks/
├── tests/
├── pyproject.toml
├── Makefile
└── README.md
```

The organizational principle of this structure is the **separation of project components**. The subfolders within `src/` correspond to the phases of work: `envs` (the custom and CybORG environments, separated), `agents` (agents ranging from Q-learning to deep RL), `training`, `evaluation`, and `explain` (policy explainability). The `dashboard` folder resides independently. Reflecting the nature of this project, there is a `docs/scope_and_limits.md` file dedicated to its research nature and the sim-to-real gap: documenting the true scope and limitations of the project displays the rigor and understanding of what has been built. Just as in previous projects, you reuse the foundation of best practices (centralized configuration, tests, Docker, and Makefile), which accelerates development and brings consistency to your portfolio.

---

## In Summary

Before writing any code, the core objectives are clear: you will build an **agent that learns, through reinforcement, to defend a network** against attacks within the paradigm of autonomous cyber defense. A simulated attacker attempts to compromise the network, and your blue agent learns, through interaction, an effective defense policy; you then evaluate, explain, and visualize it. You have grasped the central new concept (RL as the observe-act-learn loop in a responsive environment), the most important principle (that RL is suited for sequential response rather than classification, which would be supervised learning in disguise), and the honest framework the project requires (its research focus, the sim-to-real gap, and the deliberate emphasis on defense). Your stack combines standard RL tools with the reference environment, beginning with simple concepts to build a solid foundation. Finally, you have an architecture that aligns with the project components and leverages your existing best practices.

A common thread connects these elements: every decision aims to ensure that when others review your work, they conclude you have a solid grasp of reinforcement learning, apply it to a cutting-edge problem, and thoroughly understand what you are doing, including its scope. With these foundations established, you are ready to begin Phase 0 knowing not only what you will do, but also why RL is the right tool and how to present it transparently.
