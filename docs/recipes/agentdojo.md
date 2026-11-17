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

The first EvoSafe comparison has completed; SHE has completed its 20 TRAIN rounds
and its frozen TEST evaluation is still running. The completed EvoSafe reference
measurement is:

| Adapter | Clean utility | ASR | Errors / planned |
|---|---|---|---|
| None | 47/54 (87.0%) | 81/344 (23.5%) | 8/406 |
| Initial EvoSafe | 49/55 (89.1%) | 57/344 (16.6%) | 7/406 |
| Final EvoSafe | 48/55 (87.3%) | 8/347 (2.3%) | 4/406 |

The final adapter improves observed attack resistance and safe completion while
clean utility drops by one task. Errors remain errors, not successful defenses.
See the [full EvoSafe record](https://github.com/Infini-AI-Lab/hermit/blob/main/examples/agentdojo/evosafe/README.md#results)
for paired changes, selection, source/data hashes and limitations. No statistical
significance or complete-run objective is claimed.

The committed recipes require the existing learner sandbox. The current local
host blocks bubblewrap; as with the AgentDyn local experiments, explicitly
recorded `--no-sandbox` reference runs retain Docker isolation for learned hooks
but weaken the learner boundary. Those numbers must remain labeled reference
measurements, not fully sandboxed reported results. No core sandbox change is
introduced. Official AgentDojo and AgentDyn install the same `agentdojo` package
name and need separate Python environments.
