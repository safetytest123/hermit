# EvoSafeHarness on AgentDojo

[`run.yaml`](run.yaml) connects the existing EvoSafeHarness method and
`evosafe@1.0` adapter to official AgentDojo's frozen replay task.
Follow [installation and data preparation](../README.md) first. The config's
paths are relative to this folder; preparation writes `../utils/data/`.

The victim defaults to `openai/gpt-4o-mini-2024-07-18`. Configure its backend and
credentials, and replace `llm: null` with your proposer before training. The
existing EvoSafe Designer/Criticizer/Analyzer roles, candidate selection and
clean-utility constraints are unchanged. The defense's structured hook framework
is `agentdojo`; server is the native suite name. Tool schemas arrive at runtime.

The bridge enforces `episode_timeout_s: 900` independently of `agent.timeout_min`.
`max_iters: 15` bounds the native tool loop. Learned hooks require Docker and the
`python:3.12-slim` image. Keep sandbox enforcement enabled for learner execution.

After configuring the models:

```bash
hermit check --config examples/agentdojo/evosafe/run.yaml
hermit train --config examples/agentdojo/evosafe/run.yaml
hermit eval --config examples/agentdojo/evosafe/run.yaml --adapter final
```

For a no-defense comparison use `hermit eval --config ... --adapter none` on the
same frozen test IDs. The shared AgentDyn `report.py` consumes the same per-case
metric contract; it is also usable for AgentDojo run records. Failed executions
are errors, not successful defenses. No training or held-out result is claimed.

## Matching Lightning/Codex experiment

[`run_nemotron_codex.yaml`](run_nemotron_codex.yaml) uses the shared
[57/54/406 protocol](../utils/DATA.md) and the same fixed Lightning actor settings
as SHE. TRAIN has 57 cases; reserved validation is unused. The Designer,
Criticizer, Cascade and Analyzer use the existing `evosafe` implementation, with
three proposals, cascade sizes 12/57, source gates, 0.07 improvement/tolerance
settings, conservative parent-error bounds and worst-case candidate-error bounds.
Fresh Codex roles use GPT-6 Sol/high, web search disabled and read-only sessions.
The optional output classifier has four bounded same-victim calls per episode.
These are AgentDyn's existing Codex settings, not a new search variant.

```bash
python examples/agentdojo/utils/experiment.py \
  --config examples/agentdojo/evosafe/run_nemotron_codex.yaml
```

See [shared setup and sandbox limits](../README.md#shared-she-and-evosafe-comparison).
The runner exports `none`, `root` and `final` TEST comparisons automatically.
Source/data/config snapshots, per-case records and adapter exports stay local
under the ignored run's `artifacts/`. Test outcomes never feed candidate selection.

## Results

The first trained AgentDojo EvoSafe comparison is not yet complete. Tables will
report clean utility, ASR, injected utility, safe completion and errors for all
three adapters, with exact denominators. A local learner run with sandbox disabled
is reference-only and must carry that boundary exception with its scores.
