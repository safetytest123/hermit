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
