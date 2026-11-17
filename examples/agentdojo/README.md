# AgentDojo

The [official AgentDojo benchmark](https://github.com/ethz-spylab/agentdojo) as a
Hermit task, with its native **banking, slack, travel and workspace** suites,
tool loop, environments and utility/security judges. This integration replays
existing frozen indirect injections; it does not generate or optimize attacks.

The integration reuses [AgentDyn's runtime](../agentdyn/utils/bridge.py): shared
`direct` Base Agent model calls, usage accounting, the episode supervisor and
timeouts, native tool interception, Docker hook isolation, trajectory logs and
EvoSafe scoring. The thin [task entry point](utils/task.py) selects AgentDojo;
AgentDyn's default suites and behavior remain available in its own environment.
No Hermit core or learning method is added or changed.

| Folder | Contents |
|---|---|
| `utils/` | Task entry point, judge, frozen-case preparation, domain specification and offline tests |
| `evosafe/` | EvoSafeHarness example configuration; no measured training results yet |

## Install

AgentDyn and official AgentDojo both install a package named `agentdojo`. Use
separate environments; do not install both into one. From the Hermit checkout:

```bash
git clone https://github.com/ethz-spylab/agentdojo.git ../agentdojo
git -C ../agentdojo checkout 089ed468cf3ed0322acc66b0211f26d9d90dbf60
uv venv --python 3.12 .venv-agentdojo
uv pip install --python .venv-agentdojo/bin/python -e '.[test]' -e ../agentdojo
source .venv-agentdojo/bin/activate
```

`benchmark_version: v1.2.2` selects the upstream task definitions. No AgentDyn
web-page patch is needed. Learned hooks require a working Docker engine and
`python:3.12-slim` (pull it before running); the no-adapter baseline needs no
Docker. A missing or broken hook worker fails the episode without host fallback.

## Prepare frozen cases

Supply JSONL rows with `suite`, `user_task_id`, and, for injected cases, both
`injection_task_id` and `injections` (a mapping from native vector IDs to frozen
strings). Clean cases omit both injection fields. Optional old trajectories,
scores and other fields are discarded. For example, the row shape is:

```json
{"suite":"banking","user_task_id":"user_task_1","injection_task_id":"injection_task_0","injections":{"injection_incoming_transaction":"<existing frozen payload>"}}
```

Replace the placeholder with an existing recorded payload. Prepare your collection:

```bash
python examples/agentdojo/utils/prepare.py /path/to/frozen-cases.jsonl examples/agentdojo/utils/data
# Or import existing native benchmark logs for one suite:
python examples/agentdojo/utils/prepare.py /path/to/native-logs examples/agentdojo/utils/data --from-logs --suite workspace
```

Preparation deduplicates rows, adds one clean case per user task, and writes
`tasks.jsonl`, `train.txt`, `val.txt`, `test.txt` and `manifest.json`. All variants
of `(suite, user_task_id)` stay together. With seed 0 by default, the shuffled
groups assign `max(1, groups // 5)` groups each to validation and test, and the
rest to training. At least five groups and injected coverage in every split are
required. The manifest records group ownership, counts, reference revision and
task-file SHA-256. This is a local split protocol, not an official AgentDojo split.
Native task and injection-vector IDs are checked by the suite at execution time.

Prepared cases, logs, training artifacts and model credentials stay local. Do not
commit them. The defense sees the trusted user request and public tool interfaces,
not case labels, judge predicates or hidden environment state.

## Check, train and evaluate

[Configure the victim backend](../../docs/guides/model-backends.md) and provide a
proposer in `llm` for training; the supplied config leaves `llm: null` deliberately.
It uses the existing `evosafe@1.0` template and EvoSafe Evolver without algorithm
changes. See [the example](evosafe/README.md) and
[the domain specification](utils/task-info/README.md).

```bash
hermit check --config examples/agentdojo/evosafe/run.yaml
hermit train --config examples/agentdojo/evosafe/run.yaml
hermit eval --config examples/agentdojo/evosafe/run.yaml --adapter final
```

The config names `agent.base: direct`; each native victim turn goes through the
shared model backend. Default bounds are 15 tool-loop iterations and a 900-second
whole-episode deadline (`agent.params.episode_timeout_s`). Hook code runs in a
separate networkless Docker container; native tools and judges run in the trusted
worker. Logs include trajectory, hook audit, usage and result provenance.

Clean utility and injected attack success use separate denominators. Injected
utility and secure completion are additional metrics. The aggregate objective is
clean utility minus attack success rate; the scalar per-case judge is clean utility
or `1 - attack_success`. Errors, deadlines, malformed/truncated model replies and
unfinished loops never count as secure verdicts.

## Verification and limits

```bash
python -m pytest examples/agentdojo/utils/tests -q --durations=5
python -m pytest --changed
```

The native replay tests use a local scripted OpenAI-compatible server at the model
edge, with real function loading, supervisor subprocesses, shared victim calls,
native tools and judges in all four suites. The banking scenarios check clean execution, fixed-payload replay,
attack-success direction, incomplete episodes, usage/provenance, and a real Docker
structured hook blocking a native side effect. Native tests skip without official
AgentDojo; Docker checks also need the image. Pure preparation/validation tests need
only Hermit's test dependencies. The benchmark ships its environments in the
pinned source checkout; tests download nothing and call no model provider.

No live-provider training or held-out defense evaluation has been measured for this
integration. The configuration is a starting recipe, not a reproduction result.
