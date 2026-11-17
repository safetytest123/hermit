# AgentDojo defense comparison

The official AgentDojo banking, Slack, travel and workspace suites share native
replay, model calls, Docker defense hooks and scoring with the AgentDyn
integration. [Setup and recipes](https://github.com/Infini-AI-Lab/hermit/tree/main/examples/agentdojo)
cover both [SHE](../methods/she.md) and [EvoSafeHarness](../methods/evosafe.md).
No model weights, native tools or judge predicates are changed.

The frozen comparison imports existing `tool_knowledge` payloads from the pinned
upstream logs, removes prior trajectories/scores, audits native initialization,
and splits by user task. It has 57 TRAIN, 54 reserved validation and 406 TEST
cases. Both Codex recipes select on TRAIN only and reserve TEST for frozen
no-defense/initial/final evaluation. See the
[data protocol](https://github.com/Infini-AI-Lab/hermit/blob/main/examples/agentdojo/utils/DATA.md)
for source revision, exclusions, ownership and hashes.

Both methods use the same fixed Nemotron 3.5 Lightning victim, 25 tool iterations,
zero temperature and GPT-6 Sol/high Codex learners. SHE retains its 20-round
upstream measured-best selection; EvoSafe keeps its three-proposal source cascade.
Their search and auxiliary-judge budgets differ as documented method settings.
Clean utility, attack success, raw injected utility, safe completion and execution
errors have separate denominators. The scalar evaluation score is not the full
defense objective.

## Results and execution boundary

The first trained AgentDojo comparison has not completed yet. The method READMEs
will record measured adapter identity, search status, exact denominators and
paired changes when frozen TEST evaluation finishes. Scripted/offline checks do
not establish defense performance.

The committed recipes require the existing learner sandbox. The current local
host blocks bubblewrap; as with the AgentDyn local experiments, explicitly
recorded `--no-sandbox` reference runs retain Docker isolation for learned hooks
but weaken the learner boundary. Those numbers must remain labeled reference
measurements, not fully sandboxed reported results. No core sandbox change is
introduced. Official AgentDojo and AgentDyn install the same `agentdojo` package
name and need separate Python environments.
