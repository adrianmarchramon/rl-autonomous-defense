# Phase 5 — Evaluation Against Adversaries

> In this phase, you subject your agent to the moment of truth, and the question that truly matters is not "does it defend?", but rather "does it defend *really well*, and *for the right reasons*?". This phase is what separates an agent that *looks* good from one that actually is, and it embodies the commitment to honesty that runs throughout the project. It features a centerpiece that sets it apart: the **generalization** test. Remember that you trained mostly against one attacker (B_line); the big question is whether your agent learned to defend *in general*, or only to counter the specific pattern of that attacker. If the latter, it will fail against a different attacker, and this is crucial, because in real-world security, attackers never follow the training script. Checking if your policy generalizes, instead of boasting about good numbers against the known attacker, is precisely what demonstrates rigor and honesty—the same sensitivity you applied to the unimodal bias test in your disinformation project.

**Phase Objective:** honestly measure how well the agent defends.  
**Duration:** weeks 6-7.  
**By the end, you will have:** a rigorous evaluation of your agent: how well it defends against different attackers (including unseen ones) compared to baselines, along with an analysis of when and why it fails, and an honest understanding of its limitations.

---

## The Big Picture

The workflow of this phase evaluates with rigor and honesty:

```
   The Agent (Phase 4)
        │
        ├──► [ 1. Against different attackers ]  ──►  B_line, Meander, Sleep (seen and unseen)
        │
        ├──► [ 2. The generalization test ]      ──►  Does it defend in general, or only against the known?
        │
        ├──► [ 3. Compare with baselines ]       ──►  Does it beat random, passive, and a heuristic?
        │
        └──► [ 4. Analyze failures ]            ──►  When and why does it lose control?
        │
        ▼
   [ Honest evaluation + known limits ]
```

---

## Step 1 — Evaluate Against Different Attackers

The first step is to stop evaluating solely against the attacker you trained with, and measure performance against **all** attackers in the environment: B_line (the direct one), Meander (the explorer), and Sleep (the passive one, a baseline). With `evaluate_policy`, this is straightforward:

```python
from stable_baselines3.common.evaluation import evaluate_policy

from src.envs.cyborg_wrapper import make_cyborg_env

for attacker in ["B_line", "Meander", "Sleep"]:
    env = make_cyborg_env(red_agent=attacker)
    mean, std = evaluate_policy(model, env, n_eval_episodes=100)
    print(f"Against {attacker}: {mean:.1f} (±{std:.1f})")
```

Evaluating over enough episodes (one hundred or more) is important because there is randomness in the environment, and you want a stable metric, not the result of a lucky run. What you get is a profile of the agent: how it behaves against each type of attacker. And that profile is precisely what you need to address the question of generalization.

---

## Step 2 — The Generalization Test

This is the centerpiece of the phase, and it is worth looking at closely because it is what truly measures the agent's quality. The question: does your agent, trained against B_line, also defend well against Meander, which it did not see during training? If its performance against B_line is good but plummets against Meander, the conclusion is that it **overfitted**: it learned to counter the specific pattern of B_line, not to defend in general. This is a serious problem, because a defender that only knows how to stop an attacker it already knows is of little use against a real-world adversary who will do something different.

In RL, this is the equivalent of the train-to-test generalization that you know from your other projects, and it is just as fundamental here. It should also be interpreted with nuance, which demonstrates discerning judgment: Meander is intrinsically a harder attacker to defend against than B_line (the challenge's own baseline limits reflect this, showing much worse results against Meander than against B_line). Therefore, part of the drop in performance is because the problem is inherently harder, and part of it is because the attacker is new. Separating these two effects (does performance drop because Meander is more difficult, or because your agent only knows about B_line?) is the kind of honest analysis that distinguishes a serious evaluation.

A practical conclusion that evaluation often prompts: if the agent does not generalize, one way to improve it is to **train against multiple attackers at the same time** (mixing them during training), so it learns to defend against diverse patterns rather than just one. The evaluation is what tells you whether this is necessary.

---

## Step 3 — Compare with Baselines

A rigorous evaluation does not look at the agent's numbers in a vacuum, but rather in comparison to alternatives, to understand if what it has learned is truly worthwhile. It is best to compare against three baselines of increasing difficulty. The first is a **random** defender (random actions); the second is a **passive** defender (doing nothing, the blue Sleep agent). These two are easy to beat, and if your agent does not outperform them comfortably, something is very wrong. The third is the interesting and most demanding one: a **rule-based** defender—a simple, hand-written heuristic, such as "if compromise is detected on a host, restore it."

```python
def rule_based_defender(obs):
    """Simple heuristic: restore hosts that show signs of compromise."""
    ...   # returns the action to restore the suspicious host, or monitor if there are none
```

This third comparison is what truly tests your agent's value, and asking this question demonstrates maturity: does your RL-learned policy beat a reasonable hand-written heuristic? If it does, you have shown that RL contributed something that a simple rule cannot capture (a more subtle strategy). If it does not, that is an honest and important conclusion: for this problem, perhaps a simple rule is enough, and the complexity of RL was not justified. Either answer is valuable, and having the rigor to ask this question (was the sophisticated method truly worth it compared to the simple one?) is what distinguishes serious work—just as comparing with unimodal baselines did in your disinformation project.

---

## Step 4 — Analyze Failures

Beyond numbers, an honest evaluation looks at *when* and *why* the agent fails, revealing its limitations and often suggesting how to improve it. Examine the episodes where the agent performs poorly and look for patterns. Does it fail mostly against Meander, which would confirm overfitting to B_line? Does it lose control at a specific moment, for instance, when the attacker reaches a certain key host? Does it react too late, allowing the attacker to escalate to root (where, recall from Phase 2, Remove no longer works and only the costly Restore remains)? Each pattern tells you something about the limits of the strategy the agent has learned.

For example, discovering that the agent defends well while the attacker is far from Op_Server0 but collapses when it approaches shows you where its weak point lies, and perhaps how to fine-tune the reward or training to reinforce it. This analysis turns "my agent gets this score" into "I understand how and why my agent defends and fails," which is a much more valuable understanding and the one that truly demonstrates mastery of the problem.

---

## Step 5 — Interpret Honestly

The final step is to interpret and communicate all of this with the honesty the project demands, because how you present the evaluation says as much about your maturity as the results themselves. Report performance against *all* attackers, not just the one it performs best against. Be clear about **generalization**: if your agent generalizes well, state it with pride; if it overfitted to the training attacker, acknowledge it, because that honesty is worth more than a favorable number. Communicate how it compares to the baselines, including the heuristic. And frame everything by remembering the nature of the project you established in Phase 0: this is a research demonstration in a simulated environment, and the **simulation-to-reality gap** means that, no matter how well the agent defends in CybORG, it would not be a deployable defender in a real-world network. This caution does not diminish its value; it shows that you understand the actual scope of what you have built.

This honesty, as in your other projects, strengthens the work instead of weakening it. An agent presented with a rigorous evaluation that acknowledges its limits (its imperfect generalization, the sim-to-real gap) conveys much more credibility than one for which only the best numbers are shown. It is the same lesson you cultivated in the medical and disinformation projects: in serious problems, honesty about limitations is a strength.

---

## Verification: The "Definition of Done"

The phase is finished when the following are met:

- [ ] You have evaluated the agent against different attackers (B_line, Meander, Sleep) over sufficient episodes.
- [ ] You have verified generalization: how it behaves against an attacker it did not see during training, separating intrinsic difficulty from overfitting.
- [ ] You have compared it against baselines (random, passive, and especially, rule-based).
- [ ] You have analyzed failures and understand when and why the agent loses control.
- [ ] You interpret results honestly, acknowledging imperfect generalization and the sim-to-real gap.
- [ ] **The key test:** you know how well your agent defends, against which attackers, compared to alternatives, and you know its limits.

The key test has several facets that reflect the depth of this phase: knowing real performance (against multiple attackers), having verified whether it generalizes (not just whether it defeats the known), having compared it with alternatives (including a simple heuristic), and knowing its limits. The fact that all of these matter is characteristic of a serious evaluation: a good score against the training attacker is not enough; we need to know if that score is honest, if it holds up against the unknown, if it outperforms the simple, and where it breaks down. That is the difference between boasting about an agent and truly understanding it.

---

## Deliverables and What Comes Next

Upon wrapping up Phase 5, you have something that few RL projects show: a rigorous and honest evaluation of your agent. You know how well it defends against each attacker, whether its policy generalizes or overfitted, how it compares to alternatives (including a heuristic), and where its limits lie. You have applied the evaluative rigor cultivated in your previous projects and honored the commitment to honesty that frames this project, including the simulation-to-reality gap. You have an agent you can trust because you know exactly what it can and cannot do.

The next step, **Phase 6**, opens the black box: **explaining and visualizing the policy**. Your agent defends, but its strategy is hidden within the weights of a neural network, and a defender whose behavior is not understood cannot be trusted. You will analyze what strategy the agent discovered (what actions it takes in which situations), apply explainable RL techniques to make it interpretable, and build a step-by-step visualization of the agent defending the network. This is where the concept of explainability from your medical project reappears, now applied to an RL policy. You have moved from "I know how well my agent defends" to being on the verge of "I understand and can demonstrate *what strategy* it learned and why."
