# Intrusion Detection with RL — Roadmap Closure: Mistakes, Improvements, and Resources

> With all eight phases complete, you have the agent built, trained, rigorously evaluated, explained, and presented. This document covers everything surrounding the project to elevate it: the common mistakes to avoid (cross-cutting lessons for the entire project), ways to go further if you want to keep growing it, resources for deeper learning, and a final summary of the journey. The checklist for the definitive README was already developed in Phase 7, and the reflection on how this project completes your portfolio of six pieces (covering the major families of AI) was also addressed there, so they are not repeated here.

---

## Common Mistakes to Avoid

These errors are cross-cutting across the entire project, and each one holds a key lesson that distinguishes a mature RL project from a naive one. The first one, in particular, defines the project's identity.

**Using RL to classify intrusions.** This is the root conceptual mistake, which is why we addressed it starting in Phase 0: framing detection as a classification problem (each flow is a state, classifying is the action) turns RL into disguised supervised learning, stripped of the sequential structure that makes it valuable. The solution, which underpins the entire project, is to focus on **sequential response**: the agent learns to *defend* (a policy), not to *label*. Avoiding this trap is what demonstrates that you understand what RL is actually for.

**Starting directly with CybORG.** RL is difficult, and jumping straight into a realistic environment without understanding the fundamentals is a recipe for failure. The solution—the pedagogical decision made in Phase 1—is to start with a **simple environment and custom Q-learning** implemented by you, to truly understand RL before facing the complexity. This is the difference between understanding the technique and merely running a library.

**Neglecting reward design.** In RL, the reward *is* the task definition, so a poorly designed one ruins learning (or teaches the agent to cheat). The solution, which is the heart of Phase 4, is careful **reward shaping**: designing signals that guide the agent toward actual defense while keeping an eye out for reward hacking.

**Evaluating only against the training attacker.** If you only measure the agent against the attacker it already knows, you won't know if it learned to defend in general or if it just learned to counter a single pattern. The solution, the centerpiece of Phase 5, is to evaluate against **unseen attackers** and **baselines** (including a heuristic) to verify if the policy generalizes and if it truly outperforms simple approaches.

**Presenting the policy as a black box.** A security agent whose strategy cannot be understood cannot be trusted, audited, or improved. The solution, which is the objective of Phase 6, is to **explain and visualize** the learned policy, turning an opaque neural network into an understandable strategy.

**Overstating the scope.** A policy learned in a simulator is not a deployable defender on a real network. The solution, established since Phase 0, is **honesty**: making it clear that this is a research demonstration, that a simulation-to-reality gap exists, and that the focus is deliberately defensive (due to dual-use concerns).

The common thread of these mistakes is twofold. On the one hand, the conceptual aspect: the value of RL lies in the sequential defense problem, in the reward design, and in an evaluation that verifies generalization, rather than simply applying the latest trendy algorithm. On the other hand, maturity: several of these mistakes are ways to deceive oneself (or anyone looking at the work) about what the agent can actually do, and avoiding them is part of approaching the project with seriousness.

---

## Stretch Goals: If You Want to Go Further

Once you have completed the project, these extras can take it to the next level and demonstrate an even deeper mastery of RL. Each represents a recognized direction; adding one or two well-executed goals will set your work further apart.

**Training the attacker too (adversarial RL and coevolution).** Instead of using fixed attackers, you can have the red agent *also* learn via reinforcement learning, creating a coevolutionary dynamic where the attacker and defender improve against each other, resembling an arms race. This is valuable because it represents the cutting edge of autonomous defense, is more realistic (real attackers adapt), and produces more robust defenders (additionally, it directly addresses the generalization problem: a defender trained against an evolving attacker learns to generalize). It is highly difficult (adversarial and multi-agent RL are much harder due to instability and non-stationarity) but carries a high impact.

**Multi-agent (CAGE 4).** You can make the leap to the multi-agent scenario of CAGE Challenge 4, where multiple defenders coordinate to protect different areas of a network. This is valuable because real-world defense requires coordination, and multi-agent RL is a sophisticated frontier in the field. It is highly difficult (coordination between agents adds complexity) and is the natural "next level" of the environment once CAGE 2 is mastered.

**Improving generalization.** You can directly tackle the problem identified in Phase 5: training against diverse attackers and applying techniques to improve transfer to different networks or attacks. This is valuable because generalizing to unseen attackers is *the* practical limitation of a learned defender, so improving it makes the agent genuinely useful. This is of medium-high difficulty and connects directly with your evaluation findings.

**Advanced explainable RL.** You can dive deeper into explainability beyond SHAP, using techniques native to RL (reward decomposition, policy summarization, counterfactual explanations, extracting readable strategies). This is valuable because it deepens the trust and auditability established in Phase 6, and explainable RL is an active area of research, particularly in security. It is of medium-high difficulty and aligns with the explainability theme in your portfolio.

**Rigorous algorithm comparison.** You can conduct a systematic study of which RL algorithms (PPO, DQN, A2C, SAC...) perform best on this problem and why. This is valuable because it provides a small empirical contribution (which algorithm is best suited for autonomous defense?) and demonstrates experimental rigor. It is of medium difficulty and extends the PPO-DQN comparison you performed in Phase 4.

**Operational cost-aware rewards.** You can model the real-world operational costs of defensive actions (e.g., restoring a server disrupts service, isolating a machine affects users) and have the agent balance security with these costs. This is valuable because it is more realistic (in actual defense, you cannot constantly restore everything as the overhead would be unacceptable) and presents a richer objective. This is of medium difficulty (reward design) and connects with both the reward shaping of Phase 4 and the overall realism of the project.

Implementing one or two of these well will result in a standout project, especially if you choose adversarial or multi-agent RL, which are at the frontier of the field.

---

## Learning Resources

To deepen your understanding of RL and autonomous defense, here are some of the most valuable resources, along with a note on what each provides.

**An RL fundamentals book or course.** To fully master the concepts, the gold standard reference is *Reinforcement Learning: An Introduction* by Sutton and Barto. This is the theoretical foundation for everything you have done (the loop, policies, values, exploration-exploitation trade-offs). Excellent practical courses are also available if you prefer a more applied approach.

**Gymnasium and Stable-Baselines3 documentation.** For environments and algorithms, complete with plenty of examples. Useful for thoroughly understanding the standard interface and making the most of algorithm implementations (along with their tuned hyperparameters in RL Zoo).

**CybORG and the CAGE Challenges.** Their repositories and papers explain the environment and the autonomous defense problem, serving as the gateway to more advanced challenges (such as the multi-agent CAGE 4) if you wish to continue.

**The "awesome-rl-for-cybersecurity" list.** An excellent, curated compilation of resources, environments, and papers on RL applied to security. A great map of the field.

**Literature on autonomous cyber defense.** Papers on the CAGE Challenges and works on explainable RL for defense offer insight into the state of the art and where the field is heading.

**Concepts to master:** the RL loop (states, actions, rewards, policy); exploration versus exploitation; reward shaping and its pitfalls; algorithms (Q-learning, PPO, DQN) and deep RL (neural network approximation); the autonomous defense paradigm; generalization; and, most importantly, why RL is suited for sequential response rather than classification. If you can explain all of this fluently, you will demonstrate an understanding of RL that goes far beyond simply knowing how to run a library.

---

## Summary of Milestones

To recap the journey, here is the complete phase-by-phase project roadmap:

| Week | Milestone | What It Demonstrates |
|--------|------|------------------|
| 1 | Environment + Fundamentals + Approach | Foundations, understanding of RL, and the correct approach |
| 1-2 | RL Fundamentals (Simple Environment + Q-learning) | Truly understanding RL by building it from scratch |
| 2-3 | Understanding the Defense Environment (CybORG) | Managing a realistic environment and its partial observability |
| 3-5 | Agent Trained with Deep RL | Bringing the agent to life: learning to defend |
| 5-6 | Improved Agent (Reward Shaping and Tuning) | The defining skill of RL: making the agent perform well |
| 6-7 | Evaluation Against Adversaries | Proving it defends well and for the right reasons |
| 7-9 | Explaining and Visualizing the Policy | Opening the black box: understanding and showing the strategy |
| 9-10 | Packaging, README, Video, Post | Presenting the project with sound judgment and honesty |
