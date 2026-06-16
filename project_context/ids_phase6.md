# Phase 6 — Explaining and Visualizing the Policy

> In this phase, you open the black box. Your agent defends, but its strategy is hidden within the weights of a neural network: you do not know *what* it learned to do or *why*. And that is a problem, because a defender whose behavior is not understood is difficult to audit, to improve, and, above all, to trust. Here, applied to a new domain, the backbone idea of your medical project reappears: in high-stakes problems, the explanation is just as important as the outcome, and a black box is not enough. The guiding idea of this phase is twofold: to **understand** what strategy your agent discovered, and to **demonstrate** it. This has a special appeal: the agent learned its defense doctrine on its own, through trial and error, without anyone dictating it, so discovering what strategy it devised (and whether it makes sense, or if it is surprising) is genuinely fascinating.

**Phase Objective:** Understand and demonstrate the strategy the agent has learned.  
**Duration:** Weeks 7-9.  
**Upon completion, you will have:** An analysis of the strategy discovered by the agent, an interpretable explanation of its behavior (reusing techniques from your medical project), and a step-by-step visualization of the agent defending the network.

---

## The Big Picture

The flow of this phase opens, interprets, and visualizes the policy:

```
   The trained agent (black box)
        │
        ├──► [ 1-2. Analyze the policy ]  ──►  What strategy did it discover? (actions, situations)
        │
        ├──► [ 3. Explainable RL (SHAP) ]  ──►  What signals guide its decisions? (reuses XAI)
        │
        └──► [ 4. Visualize ]  ──►  See the agent defend the network step-by-step (dashboard)
        │
        ▼
   [ A comprehensible and demonstrable strategy ]
```

---

## Step 1 — Why Explain an RL Policy

It is worth establishing the motivation, as it links the project to one of the most important topics in AI and to your own prior work. Your agent's policy is opaque: a neural network that, given a network observation, produces an action without explaining its reasoning. For a security system, this opacity is a serious issue, for the same reasons it was in your medical project: you cannot trust a defender whose logic you do not understand, you cannot audit whether its strategy is sensible or has blind spots, and you cannot improve it effectively if you do not know what it is doing. That is why RL explainability (*Explainable RL*) is an active area of research, particularly for autonomous defense.

There is also a specific challenge that makes this problem interesting: explaining an RL policy is harder than explaining a single prediction, because a policy is a *behavior over time*—a strategy—not just a single decision. And there is a unique reward: because the agent discovered its strategy on its own, explaining it means finding out what defense "doctrine" an AI devised while learning independently. This strategy could be sensible, surprising, or even reveal flaws (sometimes RL agents discover shortcuts that exploit environmental quirks instead of truly defending). Opening that black box is, for all these reasons, both fascinating and valuable.

---

## Step 2 — Analyzing the Learned Policy

The first and most direct step is to analyze what the agent does: what actions it takes and in which situations, to infer its strategy. A good starting point is to observe its behavior over many episodes:

```python
from collections import Counter

import numpy as np

actions = []
for _ in range(50):
    obs, _ = env.reset()
    done = False
    while not done:
        action, _ = model.predict(obs, deterministic=True)
        actions.append(int(action))
        obs, _, done, _, _ = env.step(action)

print(Counter(actions))   # Which actions does it use most? Does it monitor, analyze, or restore?
```

From here, you look for patterns that reveal the strategy. Does the agent monitor passively until it detects indicators and then act? Does it prioritize protecting `Op_Server0`, the critical asset? Does it prefer `Remove` (fast) or `Restore` (secure but costly), and in which situations does it choose each? Does it react early or late? By observing what the agent does in different situations, you reconstruct the doctrine it learned. This analysis itself is often revealing: you frequently discover that the agent has arrived at a reasonable strategy (for example, monitoring and aggressively restoring critical servers), and sometimes that it has found something unexpected. It also connects to Phase 5: if you see that its strategy is highly specialized to a specific attacker pattern, that would explain the overfitting you detected.

---

## Step 3 — Explainable RL Techniques

Beyond observing behavior, you apply techniques to make *why* the agent decides what it decides interpretable. Here, you can successfully leverage what you learned in your medical project: **SHAP**. The idea is to treat the policy as a function (observation → action) and use SHAP to find out which observation signals (which alerts, which hosts, which indicators) most heavily influence the agent's decisions:

```python
import shap

def policy_fn(observations):
    actions, _ = model.predict(observations, deterministic=True)
    return actions

explainer = shap.Explainer(policy_fn, sample_observations)
shap_values = explainer(observations_to_explain)
# Which parts of the observation carry the most weight when the agent decides to act?
```

Being able to reuse SHAP—the same tool you used to explain a medical model—to explain an RL policy is a great demonstration of how your portfolio skills transfer and combine. It answers questions such as "What does the agent look at to decide to restore a host?" or "What signal triggers its reaction?" Another useful technique, if you want to go deeper, is **surrogate models**: approximating the policy with an interpretable model (like a decision tree) whose logic is readable, to get a simplified, legible version of the agent's strategy. With these techniques, you turn an opaque neural network into a behavior you can explain in plain words.

---

## Step 4 — Visualizing the Agent Defending

The most visually striking part, and the one that communicates the project best: building a visualization that shows the agent defending the network **step-by-step**. If analyzing the policy is "understanding," this is "showing." For an RL project, it is the equivalent of the dashboard in your other projects, as well as the best material for your presentation. Using Streamlit, which you already know, you can replay an episode and show, at each step, the network status, the attacker's progress, and the action taken by the defender:

```python
import streamlit as st

st.title("🛡️ The agent defending, step-by-step")

if st.button("Run an episode"):
    obs, _ = env.reset()
    done, step = False, 0
    while not done:
        action, _ = model.predict(obs, deterministic=True)
        obs, reward, done, _, _ = env.step(action)
        # Show: the step, the network status (which hosts are compromised),
        # the agent's action, and the cumulative reward
        st.write(f"**Step {step}** — action: {action_name(action)} · reward: {reward:.1f}")
        # (ideally, a visual representation of the network with hosts colored according to their status)
        step += 1
```

Ideally, this should be accompanied by a visual representation of the network (with hosts colored based on whether they are healthy or compromised, the attacker advancing, and the defender reacting) so that you can *see* the game play out. Being able to watch the agent detect an intrusion, decide to restore a critical host, and stop the attacker in real-time and step-by-step is highly compelling to show: it turns an abstract result (a score) into a visible and understandable story. It is also what will make your presentation video memorable.

---

## Verification: The "Definition of Done"

The phase is complete when the following are met:

- [ ] You have analyzed the learned policy and can describe the strategy the agent discovered.
- [ ] You have applied explainable RL techniques (SHAP on the policy, adapted from your medical project) to understand what guides its decisions.
- [ ] You have built a step-by-step visualization of the agent defending the network.
- [ ] You can both explain (in words) and show (visually) what the agent does and why.
- [ ] **The key test:** You can explain and demonstrate what the agent does and why, turning the black box into a comprehensible strategy.

The key test is one of transparency: if you have turned an opaque policy, hidden within a neural network, into a strategy you can explain with words and show step-by-step, you have achieved what this phase aims for. This ability to open the black box (understanding and communicating what your agent does and why) is what makes a learned defender something you can trust, and it reinforces the same lesson of explainability from your medical project, now demonstrated on a very different problem.

---

## Deliverables and Next Steps

By closing Phase 6, you have opened your agent's black box: you understand the strategy it discovered on its own, you can explain what guides its decisions (using the techniques you reused from your medical project), and you have built a visualization that shows the agent defending the network step-by-step. You have turned an opaque policy into a comprehensible and demonstrable behavior, proving that the explainability skills you possess transfer from a medical model to an RL policy. You now have the complete project: an agent that defends, which you know defends well, and whose strategy you understand and can showcase.

The next and final step, **Phase 7**, highlights the value of the entire project: **packaging and presentation**. You will containerize the system, write the final README (detailing the approach, design decisions, evaluation results, and an honest assessment of its scope), record a video showing the agent in action while explaining its strategy, and write a post about what you learned. All the work from these eight phases now needs a showcase, and you have exceptional material: an autonomously defending RL agent, rigorously evaluated, whose strategy you can demonstrate in action. You have progressed from "understanding and showing my agent's strategy" to being on the verge of "presenting the entire project to clearly demonstrate your expertise."
