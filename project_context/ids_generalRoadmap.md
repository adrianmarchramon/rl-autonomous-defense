# Roadmap: Intrusion Detection System with RL

> An AI agent that learns, through reinforcement, to **defend** a network against attacks: detecting intrusions and, most importantly, deciding how to respond to them in a simulated environment where an attacker attempts to compromise systems and the agent learns to thwart them. It combines cybersecurity with reinforcement learning (RL), a more sophisticated and less common AI paradigm than supervised learning, applied to one of today's most cutting-edge problems: **autonomous cyber defense**.

**Estimated Duration:** 8–10 weeks (part-time)  
**Level:** Intermediate-Advanced / Advanced  
**Final Result:** An RL agent that learns an effective defense policy in a simulated network environment (CybORG / CAGE), trained using Deep RL, evaluated against various attackers, with its learned strategy explained and visualized.

> **A key approach decision that defines the project from the start:** applying RL to mere intrusion *classification* (looking at a network flow and labeling it as "attack" or "normal" using a static dataset) is widely criticized, and rightly so. That is essentially supervised learning disguised as RL, lacking the sequential and interactive structure that makes RL valuable. Where RL truly shines is in the **sequential** problem of *response*: how to defend a network against an evolving intrusion by taking a sequence of actions in a reactive environment. This is why this project is not "RL for classification," but rather "RL for **detection and response**": the agent learns a *defense policy*, not a label. Making this decision with sound judgment, and avoiding the easy trap, is exactly what demonstrates a genuine understanding of what RL is for.

> **Note on the project's nature:** everything takes place in a **simulated environment**, not on real networks, which avoids security risks. It is important to be transparent about two things: first, a policy learned in simulation does not transfer directly to a real network (the simulation-to-reality gap is an open problem), meaning this is a research demonstration, not a deployable defender; second, RL applied to cybersecurity is dual-use (as there are also "red" attacker agents), which is why the project deliberately focuses on **defense**.

---

## 1. What Exactly You Will Build

A defending agent that learns through reinforcement, following this diagram:

```
   Simulated red agent (attacker) ⚔️   vs   🛡️ Blue agent (defender, RL)
        │                                          │
        ▼                                          ▼
   ┌──────────────────────────────────────────────────────────┐
   │   Simulated network environment (CybORG / CAGE)           │
   │   the attacker attempts to compromise machines and move   │
   └──────────────────────────────────────────────────────────┘
        │  observation: network state              ▲  reward: did it defend well?
        ▼                                          │
   [ RL Agent ]   ──►  observes, detects intrusions, and ACTS:
                       monitor, analyze, remove malware, restore, isolate...
        │
        ▼
   [ Learned defense policy ]  ──►  evaluated, explained, and visualized
```

The core idea, and what makes this project valuable, is what you already saw in the approach note: **RL brings what supervised learning cannot, which is learning to make sequential decisions in an interactive environment**. A classifier looks at a piece of data, outputs a label, and its job is done. A defense agent, by contrast, observes the network state, decides on an action (analyzing a machine, terminating a malicious process, restoring a system, cutting a connection), observes how the attacker reacts, and decides again, over and over, pursuing a long-term goal (keeping the network healthy). This observe-act-learn loop, where the agent's actions change the course of events, is the essence of RL and the reason it is the right paradigm for defense. The agent does not learn to *recognize* attacks; it learns to *defend* against them.

---

## 2. The Use Case and Why It Stands Out

The project addresses a cutting-edge problem—autonomous cyber defense—and combines several aspects that make it particularly powerful.

**Master a sophisticated and less common AI paradigm.** Reinforcement learning is significantly more challenging and less common in portfolios than supervised learning or even LLMs. Showing that you can apply RL, especially to a complex, real-world problem, places you at a rare and highly valued level. Furthermore, it is the missing paradigm in your portfolio, adding one of the major branches of AI.

**Work at the frontier of research.** Autonomous cyber defense (using environments like CybORG and the CAGE Challenges) is a highly active research area funded by defense agencies. Working on it shows that you operate at the cutting edge of the field, rather than on textbook problems.

**Demonstrate uncommon conceptual maturity.** As mentioned, many people apply RL incorrectly to intrusion detection (turning it into a disguised classification task). Choosing the right approach (sequential response) and knowing how to explain why demonstrates that you truly understand what each tool is for—a level of maturity that distinguishes a strong AI engineer from someone who merely follows recipes.

**Have a memorable narrative.** "I trained an AI agent to autonomously defend a network against attacks" is the kind of phrase that stands out in an interview. Being able to *show* the agent defending itself and *explain* the strategy it discovered on its own is compelling.

Additionally, this project connects naturally with your previous ones: it shares the **adversarial and cybersecurity mindset** of your honeypot project (attacker vs. defender), and reuses the **explainability** concept from your medical project (understanding the policy the agent learned, which would otherwise be a black box). It is the piece that adds RL to a portfolio that already covers supervised learning, LLMs, and multimodal AI.

---

## 3. Tech Stack (Updated to 2026)

| Layer | Tool | Why This One |
|------|-------------|--------------|
| Language | Python 3.11+ with **uv** | Standard; reuses your foundation |
| Environment Interface | **Gymnasium** | The standard API for RL environments (states, actions, rewards) |
| Defense Environment | **CybORG** (CAGE Challenge 2) | The benchmark network simulator for autonomous defense |
| Simple Environment (Learning) | Gymnasium (custom environment) | A simple problem to understand RL before the complex one |
| RL Algorithms | **Stable-Baselines3** (PPO, DQN) | Solid, proven implementations of Deep RL |
| Baseline Algorithm | Tabular Q-learning | The simplest option, to understand the fundamentals |
| Visualization | matplotlib, Streamlit | Learning curves and agent visualization |
| Quality Assurance | pytest, ruff, Docker | Reuses your best practices |

A few notes on these choices, as the rationale demonstrates technical judgment:

**Gymnasium as the interface.** It is the standard API (successor to OpenAI Gym) for RL environments: it defines the common contract of states, actions, and rewards that connects any algorithm to any environment. The fact that CybORG adheres to this API is what allows you to train standard agents on it.

**CybORG and the CAGE Challenges.** CybORG (Cyber Operations Research Gym) is the reference simulator for autonomous cyber defense: it models an enterprise network where a red agent (attacker) attempts to compromise machines and move laterally, while a blue agent (defender) must detect and respond. The CAGE Challenges are a series of public challenges using CybORG with increasing complexity. For this project, **CAGE Challenge 2** (one defender against one attacker on an enterprise network) is the ideal entry point: it is the most studied and has an approachable difficulty level compared to CAGE 4, which is multi-agent and far more complex.

**Starting simple: a custom environment and Q-learning.** This is an important pedagogical decision that shows pragmatism: RL is difficult, and jumping straight into CybORG would be like learning to swim in the open ocean. That is why you will start by building a **simple, custom environment** (a toy defense problem) and solving it with the simplest algorithm (**tabular Q-learning**). This ensures you truly grasp RL fundamentals (the loop, rewards, policy) before tackling the realistic environment. It is the difference between understanding RL and just running a library.

**Stable-Baselines3 for Deep RL.** Once in CybORG, you will use Stable-Baselines3, the standard library featuring solid, well-tested implementations of Deep RL algorithms (like PPO and DQN). This saves you from having to reimplement complex algorithms and allows you to focus on the problem itself (the environment, the rewards, the evaluation).

---

## 4. Repository Architecture

```
rl-cyber-defense/
├── src/
│   ├── envs/
│   │   ├── simple_defense.py   # simple custom environment (for learning)
│   │   └── cyborg_wrapper.py   # CybORG environment wrapper
│   ├── agents/
│   │   ├── q_learning.py       # tabular Q-learning agent (fundamentals)
│   │   └── deep_agents.py      # deep RL agents (Stable-Baselines3)
│   ├── training/train.py       # training loop
│   ├── evaluation/evaluate.py  # evaluation against adversaries
│   ├── explain/policy.py       # explaining and visualizing the learned policy
│   └── config.py
├── dashboard/app.py            # visualization of the defending agent
├── models/                     # trained agents (gitignored)
├── docs/
│   ├── context.md              # autonomous defense and RL approach
│   └── scope_and_limits.md     # research nature and sim-to-real gap
├── notebooks/
├── tests/
├── pyproject.toml
├── Makefile
└── README.md
```

The structure follows the project flow: `envs` (the environments, both simple and CybORG), `agents` (the agents, from Q-learning to Deep RL), `training`, `evaluation`, and `explain` (policy explainability), with a separate `dashboard`. Notably, `docs/scope_and_limits.md` documents the research nature of the project and the simulation-to-reality gap, showing transparency and an understanding of the actual scope of what you are building.

---

## 5. Phase-by-Phase Roadmap

Each phase includes an objective, tasks, a deliverable, and "definition of done" criteria.

---

### 🔹 Phase 0 — Setup, Fundamentals, and Approach (Week 1, First Days)

**Objective:** Environment ready, understand autonomous defense and RL, and establish a realistic, transparent approach.

**Tasks:**
- [ ] Create the repository with the structure, environment using `uv`, code quality tools, and Makefile (reusing patterns from previous projects).
- [ ] Understand what intrusion detection is, what RL is, and the autonomous cyber defense paradigm.
- [ ] Document the approach (RL for sequential response, not classification) and the reasoning behind it.
- [ ] Document the research nature of the project and the simulation-to-reality gap.

**Deliverable:** Ready environment + problem understanding + documented approach.  
**Definition of Done:** You understand why RL is the appropriate paradigm for defense (as opposed to mere classification) and are clear on the project's scope.

---

### 🔹 Phase 1 — RL Fundamentals with a Simple Environment (Weeks 1–2)

**Objective:** Gain a practical, hands-on understanding of RL before moving to the complex environment.

**Tasks:**
- [ ] Build a simple, custom environment (a toy defense problem) using the Gymnasium API.
- [ ] Solve it using tabular Q-learning, implemented from scratch.
- [ ] Understand the RL loop in practice: states, actions, rewards, policy, and exploration vs. exploitation.

**Deliverable:** A Q-learning agent that solves your simple environment.  
**Definition of Done:** You understand and can explain how an RL agent learns because you have built one from scratch.

---

### 🔹 Phase 2 — The Defense Environment (CybORG / CAGE) (Weeks 2–3)

**Objective:** Set up and run the realistic network environment.

**Tasks:**
- [ ] Install and configure CybORG (CAGE Challenge 2).
- [ ] Understand the environment: the simulated network, the observation space (what the defender sees), the action space (what it can do), and the reward.
- [ ] Understand the scenario: what the red agent (attacker) does and what the blue agent's (defender) objective is.

**Deliverable:** A working and fully understood CybORG environment.  
**Definition of Done:** You understand what the defending agent observes, what actions it can take, and its goals within the realistic environment.

---

### 🔹 Phase 3 — Training the Defending Agent (Weeks 3–5)

**Objective:** Train an agent that learns how to defend.

**Tasks:**
- [ ] Connect Stable-Baselines3 (PPO or DQN) to the CybORG environment.
- [ ] Train the blue agent while monitoring learning progress (reward curves).
- [ ] Achieve an agent that shows clear improvement over time (learning to defend).

**Deliverable:** A defending agent trained with Deep RL.  
**Definition of Done:** You have an agent that has learned a defense policy and performs significantly better than it did initially.

---

### 🔹 Phase 4 — Improving the Agent: Reward Shaping and Tuning (Weeks 5–6)

**Objective:** Turn a working agent into an effective defender.

**Tasks:**
- [ ] Apply **reward shaping**: design the reward function to better guide learning (a key leverage point in RL).
- [ ] Tune hyperparameters and, optionally, compare algorithms (PPO vs. DQN).
- [ ] Iterate until achieving a genuinely effective defender.

**Deliverable:** An improved, fine-tuned defending agent.  
**Definition of Done:** Your agent defends the network effectively as a result of careful reward design and parameter tuning.

---

### 🔹 Phase 5 — Evaluation Against Adversaries (Weeks 6–7)

**Objective:** Fairly and rigorously measure how well the agent defends.

**Tasks:**
- [ ] Evaluate the agent against different red agents (attackers), including some unseen during training.
- [ ] Compare against baselines (a random defender, a rule-based defender).
- [ ] Analyze when and why it fails, and whether its policy generalizes well.

**Deliverable:** A rigorous evaluation of the agent.  
**Definition of Done:** You know how well your agent defends against various attackers compared to baseline alternatives and understand its limitations.

---

### 🔹 Phase 6 — Explaining and Visualizing the Policy (Weeks 7–9)

**Objective:** Understand and showcase the strategy the agent learned.

**Tasks:**
- [ ] Analyze the learned policy: which actions the agent takes in specific situations and what strategy it discovered.
- [ ] Apply explainable RL techniques to make its behavior interpretable.
- [ ] Build a visualization (a dashboard) of the agent defending the network step-by-step.

**Deliverable:** An explanation and visualization of the agent's policy.  
**Definition of Done:** You can explain and demonstrate what the agent does and why, turning the black box into an understandable strategy.

---

### 🔹 Phase 7 — Packaging and Presentation (Weeks 9–10)

**Objective:** Showcase the project's value.

**Tasks:**
- [ ] Containerize the system (reusing your existing patterns).
- [ ] Write the final README: detailing the problem, approach, design decisions, results, and transparency regarding scope.
- [ ] Record a video showing the agent defending the network and explaining its strategy.
- [ ] Write a post about the project and your learnings on RL and autonomous defense.

**Deliverable:** Containerized system + high-quality README + video + post.  
**Definition of Done:** Anyone can understand the project in two minutes, grasp its sophistication (RL + cybersecurity + explainability), and see the agent in action.

---

## 6. The Ideal README (Brief Checklist)

The README should include: an engaging title and description; the problem and why it matters; an architecture diagram (attacker vs. defender); a GIF or video of the agent defending the network; the tech stack with badges; setup instructions; a **design decisions** section (why RL was chosen for response instead of classification, why CybORG, why start simple); a **scope and limitations** section (highlighting that it is research and noting the sim-to-real gap); and **results** (how well it defends against which attackers, compared to baselines). (This section can be expanded in detail later.)

---

## 7. Common Pitfalls to Avoid

| Pitfall / Mistake | Why It Is Bad | What to Do |
|-------|-----------------|-----------|
| Using RL to classify intrusions | It is supervised learning in disguise; it wastes the potential of RL | Frame it as sequential response (defense) |
| Starting directly with CybORG | RL is difficult; you risk struggling without understanding the fundamentals | Start with a simple environment and Q-learning |
| Neglecting reward design | In RL, a poorly designed reward ruins learning | Use careful reward shaping |
| Evaluating only against the training attacker | You will not know if the policy generalizes | Evaluate against unseen attackers and baselines |
| Presenting the policy as a black box | An uninterpretable security agent cannot be trusted | Explain and visualize the learned policy |
| Overstating the scope | A policy in simulation is not a real-world defender | Be transparent: it is research, and a sim-to-real gap exists |

---

## 8. Stretch Goals (Going Further)

- **Train the attacker as well (adversarial RL):** have the red agent learn simultaneously, creating a co-evolutionary attacker-defender dynamic.
- **Multi-agent (CAGE 4):** step up to the multi-agent scenario, where multiple defenders coordinate to protect different areas of the network.
- **Generalization:** study and improve how well the policy transfers to networks or attacks different from those used during training.
- **Advanced explainable RL:** apply more sophisticated techniques to understand why the agent makes each decision.
- **Algorithm comparison:** conduct a rigorous study on which RL algorithms perform best in this domain and why.
- **Operational cost-aware rewards:** model the operational cost of certain defenses (e.g., isolating a machine) and have the agent account for these trade-offs.

---

## 9. Learning Resources

- **An RL fundamentals course or book:** to master key concepts (the Sutton and Barto book is the classic reference; there are also excellent practical courses).
- **Gymnasium and Stable-Baselines3 documentation:** for environments and algorithms, complete with many practical examples.
- **CybORG and CAGE Challenges:** their repositories and research papers explain the environment and the autonomous defense problem.
- **The "awesome-rl-for-cybersecurity" list:** a comprehensive compilation of resources on RL applied to security.
- **Literature on autonomous cyber defense:** research papers from the CAGE Challenges and explainable RL in defense provide a solid view of the state of the art.
- **Key concepts to master:** the RL loop (states, actions, rewards, policy); exploration vs. exploitation; reward shaping; core algorithms (Q-learning, PPO, DQN); the autonomous defense paradigm; and why RL is suited for sequential response rather than classification.

---

## 10. Milestone Summary

| Week | Milestone | Status |
|--------|------|--------|
| 1 | Environment + fundamentals + approach | ⬜ |
| 1–2 | RL fundamentals (simple environment + Q-learning) | ⬜ |
| 2–3 | Defense environment (CybORG) up and running | ⬜ |
| 3–5 | Defending agent trained with Deep RL | ⬜ |
| 5–6 | Improved agent (reward shaping and tuning) | ⬜ |
| 6–7 | Evaluation against adversaries | ⬜ |
| 7–9 | Explain and visualize the policy | ⬜ |
| 9–10 | Packaging, README, video, post | ⬜ |

---

**The ultimate goal:** is that when someone looks at this project, they think, *"this person understands reinforcement learning, applies it to a cutting-edge security problem, and truly knows what they are doing."* It is not just another classifier: it is an agent that learns to defend a network by making sequential decisions, rigorously evaluated, with a strategy you can explain, and presented transparently regarding its scope. This is the kind of project that shows you do not just follow recipes; you understand the tools and know exactly when to use them.
