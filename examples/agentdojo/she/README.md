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

The native suites also lack SHE's signed tool registry. The reused runtime
checks membership against the native public tool schemas, while signed-registry
integrity is unavailable, exactly as in the AgentDyn task adaptation. Neither
this example nor its scores establish signed-tool integrity.

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

**Reference measurements only:** the local run uses the recorded `--no-sandbox`
override because this host cannot initialize bubblewrap. Learned hooks remain
Docker-confined, but the learner lacks the framework's host file/network boundary.
These real model measurements are not fully sandboxed reported results.

### Completed search

Run `agentdojo-she-lightning-codex-v1` completed all 20 TRAIN rounds without learner
errors. The official selector updated its best **record** at rounds 2, 6 and 9;
finalization restored the round-9 measured snapshot. Rounds 6 and 9 measured the
same four-artifact content, so their different scores are repeat variability, not
another learned artifact update. The final adapter's rule bank and tool policy
differ from seed; system prompt and safety memory remain unchanged.

The final adapter-tree training commit is
`0eac808a01855e864a1a0a9fc8138766f504c69b`. Its four-artifact digest is
`092ba06a0f22f0582c993bfb6d96099db71a3b83bf79124cd8da5c9b519fd440`, matching both the
official best harness and round 9's observed snapshot (also round 6's content).
The round-20 unmeasured pending proposal has different digest
`11b67239604f3ed63fb8bb2be5a7638950b10c9873169bfe34da185e7026970a` and is excluded.

On TRAIN, seed ASR is 7/37 and safe UA 25/37; the best round-9 record has ASR 0/37
and safe UA 33/37. Both records have one injected execution error and clean native
utility 17/19. Round 6 has the same artifact as round 9 but safe UA 32/37, showing
why best-record promotions cannot be interpreted as distinct causal improvements.
Failures are excluded by the upstream selector's completed-case denominators and
must remain separately reported. These TRAIN numbers are not held-out results.

The source is frozen at `f829fc91390d81c992fbd6a90cb56e19d8b5396c`, official AgentDojo
at `089ed468cf3ed0322acc66b0211f26d9d90dbf60` with native task version `v1.2.2`, and
SHE at `0c656460d9d8acdf406a2271d657f7a7b60bb255`. The task JSONL SHA-256 is
`d50945a77e81afb21a7556a10b29d1989a99c4822eabd45f81c4d67013f0d65e`.
The configured victim is `nvidia/nvidia/nemotron-3.5-lightning`, Codex requests
`gpt-6-sol`/high with CLI `0.164.0`, and no separate observed serving-model ID is
available from Codex exec. Pre-TEST Trainer token counters are
`rollout=24,422,520` and `evolver=3,158,175`, with 4,672.7 seconds of Trainer wall
time. Dollar prices are unconfigured; zero known-subtotal fields are not zero cost.

An independent audit of retained records/code confirmed all 1,140 learning rows
and all selection IDs belong to TRAIN, and verified the observed/pending snapshot
chain and final export. Retained SHE role events have no tool calls. This is not
proof of complete host isolation: the learner sandbox was disabled, and not every
historical `prompt.txt` is retained. The audit did not treat incomplete TEST output
as final performance.

### Frozen TEST

The 406-case none/root/final TEST comparison is still running. No held-out SHE
score is complete yet. The final table will use exact completed-case denominators,
raw utility and safe UA separately, and all planned-case errors. Local artifacts
remain under ignored `runs/agentdojo-she-lightning-codex-v1/artifacts/`.
