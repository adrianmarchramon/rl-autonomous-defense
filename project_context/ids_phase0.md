# Phase 0 — Setup, Fundamentals, and Approach

> This phase lays the foundation, and in this project, it has a distinct characteristic that must be kept crystal clear: **establishing the correct approach takes priority over writing any code**. As you saw in the fundamentals, there is an easy trap that many people fall into (using RL to classify intrusions, which is supervised learning in disguise), and the most important decision of the entire project is to avoid it: committing, from the very beginning, to a genuinely RL-based approach—that of *sequential response*. If you do not establish this now, it is very easy to drift unintentionally into that trap and waste your effort. Therefore, this phase is not just about "setting up the environment," but rather "getting the approach right." Furthermore, this is where you demonstrate from minute one that you understand what RL is actually used for, which is exactly what distinguishes a mature project from one that simply applies pre-made recipes.

**Phase Objective:** Environment ready, understanding autonomous defense and RL, and establishing an honest approach.  
**Duration:** The first few days of Week 1.  
**Upon completion, you will have:** A well-structured repository, a solid understanding of RL and autonomous cyber defense, and, most importantly, the project's approach defined and documented (RL for response, not for classification), along with its honest scope.

---

## The Big Picture

This phase is about setting the right foundation across four fronts, and note that no agent is being built yet (that begins in Phase 1): here, we prepare and make decisions.

```
   [ 1. Repository and environment ]  ──►  the engineering foundation (reused)
   [ 2. Understanding the problem ]   ──►  IDS, RL (the loop), autonomous defense
   [ 3. Establishing the approach ]   ──►  RL for sequential response, NOT classification
   [ 4. Scope and limits ]           ──►  research, sim-to-real gap, dual-use
```

---

## Step 1 — The Repository and the Environment

You begin, as in your other projects, by setting up the engineering foundation, which you already master and can reuse almost as-is. Create the repository with the structure we saw in the fundamentals: the environment using `uv`, code quality with Ruff, and a Makefile with the usual shortcuts.

```bash
uv init rl-cyber-defense
cd rl-cyber-defense
uv add gymnasium               # the environment interface, which you will use starting in Phase 1
uv add --dev pytest ruff
mkdir -p src/{envs,agents,training,evaluation,explain} dashboard models docs notebooks tests
```

A note on dependencies: in this phase, the environment is lightweight. You add `gymnasium` because you will use it immediately in Phase 1 (for your own simple environment), but you will add the heavy components (Stable-Baselines3 for Deep RL, and CybORG for the realistic environment) in the phases where they belong, to avoid installing things you do not yet need all at once. The subfolders in `src/` reflect the components of the project (environments, agents, training, evaluation, explainability). Configure `.gitignore` to exclude the `models/` folder (trained agents do not go into the repository as they are heavy files), and set up the Makefile with `make test`, `make lint`, etc. This reused foundation saves you time and brings consistency to your portfolio.

---

## Step 2 — Understanding the Problem: IDS, RL, and Autonomous Defense

Before touching anything, spend time thoroughly understanding the problem and the technique, because there is a difference here compared to your other projects: RL is a **new paradigm** in your portfolio, so truly understanding it is not a mere review, but a primary task. You cannot take shortcuts when it comes to understanding RL. Document your findings in `docs/context.md`, which will also serve as material for the README.

Make sure you are clear on the concepts we covered in the fundamentals: what **intrusion detection** is and why defense goes beyond detection (one must respond). And, above all, what **reinforcement learning** is: the loop of an agent observing a state, choosing an action, receiving a reward and a new state, and thereby learning a policy that maximizes reward over time. Internalize what makes it special: it learns through interaction and trial-and-error (not from labeled examples), its actions affect what comes next (it is sequential and interactive), and the exploration-exploitation dilemma. Additionally, understand the paradigm of **autonomous cyber defense**: the game between a red agent (attacker) and a blue agent (defender, the RL agent), where the blue agent learns to protect the network.

Properly understanding RL now is what will allow you, in subsequent phases, to design rewards thoughtfully, interpret what the agent is learning, and diagnose why it sometimes fails to learn. It is the investment that makes everything else possible.

---

## Step 3 — Establishing an Honest Approach

This is the most important step of this phase, and the intellectual heart of the project. This is where you commit, in writing, to the correct approach to avoid drifting toward the easy trap. It is highly beneficial to make the reasoning very clear, because knowing how to explain it is precisely what demonstrates maturity.

The approach is: **using RL for sequential response (defense), not for classification**. And the reason, as you already saw in the fundamentals, is as follows: if you frame intrusion detection as classification (where each network flow is a "state", "classifying as attack or normal" is the "action", and the reward is success or failure), then the agent's action does not affect what comes next. The subsequent flow is independent of the previous one; there is no sequence, no strategy, and no long-term goal. That is, quite literally, a classifier with a convoluted training process, and it completely wastes what makes RL valuable. In contrast, if you frame the problem as *defense* (the agent observes the network, decides on actions that change it, the attacker responds, and the goal is to keep the network healthy over time), you indeed have the sequential and interactive structure that RL requires and leverages.

This is why this project is framed as "detect and **respond**," rather than "classify": the agent learns a *defense policy*, not a label. Keep this documented, along with its reasoning, in the project documentation. This is not just a conceptual detail: it is the decision that will guide all others (which environment you use, how you design the reward, how you evaluate), and the one that, when well-explained, demonstrates that you truly understand what each AI tool is meant for.

---

## Step 4 — Documenting Scope and Limits

With the approach established, you also document the honest scope of the project, which is another sign of maturity. Record this in `docs/scope_and_limits.md`. There are two points that should be made clear.

The first is the **sim-to-real gap** (simulation-to-reality gap). The entire project takes place in a simulated environment (CybORG), which avoids the risks of touching real networks, but also means that a policy learned there does not transfer directly to a real network, which is far more complex, noisy, and unpredictable than any simulation. Therefore, it is advisable to make clear that this is a **demonstration of the approach**, not a defender ready for deployment. Stating this does not diminish its value; it shows that you understand the actual scope of what you have built, which is the exact opposite of overhyping.

The second is **dual-use**. RL applied to cybersecurity can also be used for offensive purposes (there are red agents that learn to compromise networks), so it is useful to make it explicit that the project deliberately focuses on **defense**. Acknowledging the dual-use nature of the technology, and positioning yourself clearly on the defensive side, is a sign of responsibility.

Documenting these limits, rather than ignoring them or exaggerating the system's capabilities, communicates that you understand the real weight and scope of your work—a level of maturity that a discerning evaluator highly appreciates.

---

## Step 5 — Recording Decisions

As in your other projects, keep a record of the key decisions in your documentation: the approach (RL for response, not classification) and its reasoning, the chosen environment (CybORG / CAGE Challenge 2) and why, the pedagogical decision to start with a simple environment using Q-learning before moving to CybORG, the planned algorithms, and the honest scope. Having this in writing will serve you well for the README, help you maintain consistency, and demonstrate that you have made deliberate decisions, particularly the decision regarding the approach that forms the backbone of the project.

---

## Verification: The "Definition of Done"

The phase is complete when the following are met:

- [ ] The repository has the structure, the environment with `uv`, code quality, and the Makefile.
- [ ] You understand RL (the loop, the policy, its sequential and interactive nature) and autonomous defense, and you have documented this in `docs/context.md`.
- [ ] The approach is defined and documented: RL for sequential response, not for classification, along with its reasoning.
- [ ] The scope and limits are documented in `docs/scope_and_limits.md` (research, sim-to-real gap, dual-use).
- [ ] You have recorded key decisions (approach, environment, pedagogical plan).
- [ ] **The key test:** You understand why RL is the appropriate paradigm for defense (and not for mere classification), and you are clear about the project's scope.

The key test has two halves that reflect the dual character of this phase: comprehension (understanding RL and why it fits sequential response) and judgment (having established the correct approach and honest scope). That the second half is so important is what is characteristic of this project: here, *getting the approach right* is just as critical as understanding the technique, because a mistaken approach (classification) would ruin the entire effort, no matter how well executed.

---

## Deliverables and What Comes Next

As you close Phase 0, you have the foundations firmly set: the engineering base is established, you have a solid understanding of RL and autonomous defense, and, above all, the project's approach has been thoughtfully defined (RL for response, not classification) along with its honestly documented scope. You have started the project exactly where a good AI engineer would: making sure to understand the tool and frame the problem correctly before writing a single line of code.

The next step, **Phase 1**, takes you to hands-on RL: the **fundamentals of RL with a simple environment**. You will build your own simple environment (a toy defense problem) using Gymnasium, and you will solve it with the simplest algorithm, tabular Q-learning, implemented by yourself. The goal is not that environment itself, but to truly understand how an RL agent learns (the loop, rewards, policy, exploration) in a scenario where you can observe everything, before tackling the realistic environment. You have moved from "I understand RL and have framed the problem correctly" to being on the verge of "I have built an RL agent from scratch and understand how it learns."
