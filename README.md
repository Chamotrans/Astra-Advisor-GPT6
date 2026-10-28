# Astra Advisor

[English](README.md) | [繁體中文（香港）](README.zh-HK.md) | [简体中文](README.zh-CN.md)

GPT-6 routing edition maintained by Chamotrans, based on [DannyMac180/astra-advisor](https://github.com/DannyMac180/astra-advisor).

**GPT-6 Astra plans the work, chooses useful bounded delegation dynamically, and
owns verification and acceptance.**

Astra Advisor is a Codex plugin for capability-routed software delivery. Give Astra
the goal, constraints, and repository context; it decides whether independent work
should run alongside the parent session and chooses a supported native subagent
model and effort for each bounded deliverable.

## Cloud limitation

ChatGPT Work cloud `create_thread` must omit `model` and
`thinking`, so it cannot currently promise arbitrary model or effort control. Astra
does not dispatch a model-pinned request there by default. Native Codex subagents are usable
where the current tool schema exposes the needed controls.

## Go deeper

Original author Daniel McAteer writes [Attention Heads](https://attentionheads.substack.com/) — deep,
evidence-backed writing on AI, cognition, and agentic engineering. The **Agentic
Engineering Field Notes** series covers the craft of using AI. [Subscribe](https://attentionheads.substack.com/subscribe?utm_source=github&utm_medium=readme&utm_campaign=astra-advisor)
to get new posts in your inbox.

## Quick start

If a marketplace named `astra-advisor` is already registered from another source,
remove that marketplace before adding this fork. Existing local installations can
continue using their local source.

Install the plugin in a current Codex CLI or ChatGPT desktop app with plugins
enabled. Start a fresh task after installation and select GPT-6 Astra at any effort
supported by the current Codex host:

~~~sh
codex plugin marketplace add Chamotrans/Astra-Advisor-GPT6 --ref main
codex plugin add astra-advisor@astra-advisor
~~~

Start a task with:

~~~text
Use $astra-advisor:orchestration to plan, build, verify, and review this work.
~~~

## How routing works

Astra remains the architect and acceptance owner in the primary GPT-6 Astra session
at the effort selected by the user. After capability preflight, Astra records the
parent model and effort as observed or unobservable before implementation or
delegation begins. The skill never changes the parent session.

When delegation helps, Astra uses the exposed generic `collaboration.spawn_agent`
tool with an explicit `model`, `reasoning_effort`, and `fork_turns: none`. It chooses
among `gpt-6-astra`, `gpt-6.1-sol`, `gpt-6.1-terra`, `gpt-6-sol`, and `gpt-6-luna` from the task's risk,
context, and independent work. There are no predefined role TOMLs, companion
installer, role-to-model mapping, or fixed subagent count cap. Astra gives each
subagent a concrete bounded deliverable and continues useful parent work while it
runs.

Live tool metadata is authoritative. The current documented effort snapshot is:

| Model | Known efforts |
| --- | --- |
| `gpt-6-astra` | `low`, `medium`, `high`, `xhigh`, `max`, `ultra` |
| `gpt-6.1-sol` | `low`, `medium`, `high`, `xhigh`, `max`, `ultra` |
| `gpt-6.1-terra` | `low`, `medium`, `high`, `xhigh`, `max`, `ultra` |
| `gpt-6-sol` | `low`, `medium`, `high`, `xhigh`, `max`, `ultra` |
| `gpt-6-luna` | `low`, `medium`, `high`, `xhigh`, `max` |

Terra 6.1 is available for scoped code fixes, visual reasoning, computer use, and
writing from supplied material. Its listed efforts were observed in Codex on
2026-10-27; API effort settings differ. Always use the live native tool schema.

If a selected model, effort, control, or tool is unavailable, conflicting, or
unobservable, Astra fails that delegation closed and reports the limitation. It does
not silently substitute a model, effort, role, or fabricated tool. Chosen values and
runtime-confirmed values are reported separately.

For substantial implementation, Astra inspects the complete diff and reruns the
requested checks, then sends the accumulated change set to a fresh read-only
reviewer. The reviewer can be any of the five supported models at a live-supported
effort. Astra accepts the work only after the reviewer returns `ship`; `fix-first`
requires a new parent verification and fresh review, while `rethink` requires a
revised plan.

## Live visibility and cost receipts (0.2.0)

Every delegation announces its name, bounded task, selected model and reasoning
effort, and selection reason. Its result reports actual status and runtime-observed
settings, or explicitly says those settings are unobservable. These updates also
cover fresh reviewers. A requested setting is not proof of the realized setting.

Every task ends with an API-equivalent cost receipt. When native tools expose token
usage, the receipt estimates its USD price using the versioned snapshot and compares
that same token workload repriced entirely at Astra. It separates whole-task,
delegated-only, and partial coverage. Missing parent or reviewer usage prevents a
whole-task claim. Without observed usage, the receipt says why it is unavailable.

The difference is a **same-token API price comparison**. It does not measure what an
all-Astra run would actually consume, actual net task savings, quality, speed, or a
change to ChatGPT subscription charges or usage credits. No subagents means no
delegation savings. Reasoning effort does not multiply the token price.

Current model rates are unavailable until a new verified pricing snapshot is supplied.
The calculator defaults to [unverified pricing status](plugins/astra-advisor/pricing/2026-10-05-unverified.json),
which contains no rates. The archived [September snapshot](plugins/astra-advisor/pricing/2026-09-04.json)
remains only for historical accounting fixtures. Do not relabel old rates for new models.

Try the clearly labeled illustrative workload (not a receipt for your task):

~~~sh
python3 plugins/astra-advisor/scripts/cost_receipt.py plugins/astra-advisor/examples/illustrative-usage.json
sh plugins/astra-advisor/scripts/verify.sh
~~~

The calculator emits JSON and accepts `--pricing PATH` for another verified snapshot.
Its input lists agents and unique atomic calls, usage provenance, coverage assertions,
and explicit pricing eligibility. It validates cached-input and reasoning-output
subsets, refuses overlapping aggregates, and keeps unknown usage separate from zero.
See the [operations reference](plugins/astra-advisor/skills/orchestration/references/operations.md)
for the input contract and receipt policy.

## ChatGPT app tasks

Separate app tasks require an explicit user request. For an explicit Codex app task,
`mcp__codex_app__create_thread` supports `model` and `thinking`; call
`mcp__codex_app__list_projects` first for project targets, use local by default, and use a worktree only when explicitly requested. Cloud `create_thread` omits both controls,
so the bounded limitation above applies. Do not use an API key, nested CLI, or
invented tool as a workaround.

## Updating

Update this fork and reinstall the plugin, then start a fresh task. Switching the
marketplace to the upstream repository replaces this edition's GPT-6 routing.

~~~sh
codex plugin marketplace upgrade astra-advisor
codex plugin add astra-advisor@astra-advisor
~~~

For local development, install this checkout as a marketplace:

~~~sh
cd /absolute/path/to/astra-advisor
codex plugin marketplace add /absolute/path/to/astra-advisor
codex plugin add astra-advisor@astra-advisor
~~~

For operational details, read
[the orchestration operations reference](plugins/astra-advisor/skills/orchestration/references/operations.md).

## Privacy and contributions

Use a public handle and your GitHub-provided noreply email for commits. Before
sharing a patch, issue, screenshot, or log, remove personal paths, contact details,
client material, credentials, and session transcripts. Examples and cost receipts
shared publicly must use synthetic data, not real session records. The ignore rules
help prevent accidental additions; they do not sanitize tracked files or Git
history. Public project attribution and the MIT copyright notice are retained.
See [LICENSE](LICENSE).
