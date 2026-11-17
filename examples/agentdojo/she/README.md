# SHE on AgentDojo

This uses the existing `she@1.0` four-artifact adapter and `she` Evolver, calling
[SHE's pinned upstream library](https://github.com/RainbowQTT/SHE/tree/0c656460d9d8acdf406a2271d657f7a7b60bb255).
The system prompt, rule bank, safety memory and tool policy evolve; the victim,
AgentDojo tasks, native judges and frozen injection payloads remain fixed.
The [method guide](../../../docs/methods/she.md) documents update/selection fidelity.
No new method or replacement selector is introduced for AgentDojo.

## Setup and protocol

Follow the [AgentDojo comparison guide](../README.md#shared-she-and-evosafe-comparison)
and [data protocol](../utils/DATA.md). Install the additional dependency in the
same resolver invocation as the benchmark and Hermit:

```bash
uv pip install --python .venv-agentdojo/bin/python -e '.[test]' --group she -e ../agentdojo
```

`run_nemotron_codex.yaml` matches AgentDyn's tested actor settings: Nemotron 3.5
Lightning, 25 tool iterations, 8,192 output tokens, temperature zero, 12 concurrent
episodes, and a 900-second whole-episode deadline. The selective judge uses the
same frozen model, with eight bounded calls per episode. Codex uses GPT-6 Sol,
high effort, web search disabled, read-only sessions and a 1,500-second role bound.
`run.yaml` is a minimal API-proposer example and needs its `llm` configured.

The comparison data has 57 TRAIN cases (19 user groups), 54 reserved validation
cases (18 groups) and 406 TEST cases (55 groups). Validation is unused by this
TRAIN-only protocol. Each of at most 20 rounds measures all TRAIN cases; upstream
`maybe_update_best` applies the `initial_ua_floor` gate. Diagnosis, attribution
and patches use current TRAIN observations only. A proposed adapter becomes
eligible after the next measured round. Finalization restores measured best,
including when the latest proposal was never measured.

AgentDojo, like AgentDyn, supplies no clean unsafe-behavior label. Clean UBR is
unavailable rather than inferred from native utility. The official score and
available-metric gate are unchanged; the gate does not guarantee clean task
utility. Report raw clean utility and its paired regressions separately. Safe UA
means native injected task success **and** attack failure; it differs from raw
injected utility. Failed executions stay out of completed-case denominators and
are counted separately. Neither failures nor missing verdicts count as defenses.

## Run

Configure `VICTIM_API_KEY` and a private `HA_BACKENDS` mapping for `vllm`, as in
the [shared guide](../README.md). Commit source/config changes and choose fresh
run/tree names before invoking the frozen runner:

```bash
python examples/agentdojo/utils/experiment.py \
  --config examples/agentdojo/she/run_nemotron_codex.yaml
```

The runner checks Docker, the seed's four artifacts and the shared Codex role;
freezes source, data and settings; trains; then evaluates `none`, `root` and
`final` on identical TEST IDs. It exports comparison reports, per-case records,
usage and adapter snapshots under the ignored run's `artifacts/` directory.
Learning failures/refusals remain recorded failures; a process exit alone does
not establish successful learning. Refusals are not rerouted or retried.

Keep learner sandbox enforcement enabled for reported results. On this host,
bubblewrap fails with `Failed RTM_NEWADDR: Operation not permitted`. A local
reference run may use the existing `--no-sandbox` runner option, recording that
override while retaining learned-hook Docker confinement. Such scores are
reference measurements with a weaker learner boundary, not fully sandboxed
reported results. See [the shared execution boundary](../README.md#execution-and-results).

## Results

The 57/54/406 protocol and real-model preflight are being prepared for the first
comparison. No trained AgentDojo SHE result has been completed yet. The final
results will distinguish `none`, initial seed and measured-best final adapter,
with exact utility/ASR/safe-UA denominators, errors, learning status and provenance.
