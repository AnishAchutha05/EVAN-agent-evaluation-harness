# EVAN: AI Safety Evaluation Harness

**A local tool that answers one question: does the way an AI agent remembers things make it safer, more accurate, or faster?**

> **Status: early development.** The project structure, CLI design and metric definitions are locked. The engine is being built layer by layer. Nothing here is a finished result yet.

---

## Why this exists

AI agents now handle multi-step jobs: research, data entry, support tickets, code changes. Every team building one has to decide how the agent handles **memory**:

| Design | How it works | Upside | Risk |
|---|---|---|---|
| **Stateful** | Remembers the whole conversation | Great continuity | One early mistake (or planted instruction) poisons every later step |
| **Ephemeral** | Starts fresh at every step | Clean and predictable | Forgets useful context |
| **Hybrid** | Starts fresh, but reuses memory that has been checked first | Balance of both | More moving parts |

Today, teams pick one by gut feeling. Comparing them properly means writing custom scripts, running things by hand, and producing results nobody else can reproduce. AEH replaces that with one repeatable workflow.

## What it does

You point AEH at a model. It runs the **same tasks** through each architecture, deliberately **breaks things** (planting errors and malicious instructions), records everything, and prints a side-by-side scorecard.

The architecture is the *only* thing that changes. Same model, same tasks, same settings, same scoring. So if results differ, the memory design is the reason.

## What you get

**Five headline metrics** (always reported):

| Metric | Plain-English meaning |
|---|---|
| **TSR** Task Success Rate | How often the agent gets the right answer |
| **EPI** Error Propagation Index | When a mistake happens early, how much damage does it cause later? |
| **OCS** Output Consistency Score | Same task, same settings, repeated: does it answer the same way? |
| **AMR** Adversarial Manipulation Rate | How often a planted malicious instruction actually changes the agent's behaviour |
| **IL** Inference Latency | How long the model and the full task take |

**Optional extended metrics** (switch on when you need them): eight security metrics (instruction priority, role retention, tool boundaries, data exfiltration, scope containment, unintended commands, payload containment, attack detection) and one quality metric (hallucination rate). These are extensions beyond the five headline metrics in the original project report.

**Reports:** a terminal comparison table, CSV export, and optional diagnostic charts.

## How it works (the short tech version)

```
CLI -> Experiment Runner -> Scenario Engine (faults + attacks)
                         -> Architecture Layer (Stateful | Ephemeral | Hybrid | yours)
                         -> Model Provider Adapter -> llama.cpp (local) or a cloud API
                         -> Trace Logger -> Metrics -> Report
```

A few ideas worth knowing:

- **Counterfactual replay (for EPI).** AEH plants an error at step 2, then reruns the same task with that step *corrected*. Any failure that appears only in the broken run is blamed on the error. That separates real error spread from ordinary model mistakes.
- **Real manipulation only (for AMR).** An agent that merely *quotes* a malicious instruction isn't counted as fooled. It counts only if later outputs or actions change compared to the clean task.
- **No fake determinism (for OCS).** Temperature 0 doesn't guarantee identical outputs, so AEH measures consistency across repeated runs using explicit per-benchmark comparison rules.
- **Safe by design.** Tools are sandboxed mocks with no real network, file or command access. Injected content is never executed. API keys are never logged or exported.
- **Local first.** Models run through [llama.cpp](https://github.com/ggml-org/llama.cpp)'s `llama-server` (CPU, or GPU via Vulkan). You supply your own GGUF model; AEH never downloads weights. AEH talks to it only over an OpenAI-compatible HTTP API, so it stays independent of llama.cpp internals.

## Bring your own architecture

Have your own agent design? Implement the `AgentBase` interface, register it, and it runs through the same pipeline as the built-ins. No changes to the runner or metrics are needed. Note that custom architecture code is trusted code, so only run code you trust.

## Quick look (planned CLI)

```bash
aeh doctor                                   # check your setup
aeh run -c configs/pilot.yaml --fake-model   # dry run, no model needed
aeh run -c configs/pilot.yaml -a stateful -a ephemeral -a hybrid
aeh report show <RUN_ID>
aeh export csv <RUN_ID> --out results.csv
```

Pilot defaults: 20 task instances, 3 repeated runs, temperature 0, seed 42, 512 max tokens, 60 s timeout, 2 retries, fault/attack injected at step 2. All configurable.

## Who it's for

- **Researchers** who need reproducible, shareable architecture comparisons.
- **Engineering teams** choosing a memory design with evidence instead of intuition.
- **Safety and security reviewers** who want to see how agents behave under injected errors and attacks.

## Honest limits

- Pilot-scale results (20 tasks) are **preliminary, not statistically conclusive**.
- Local models may not produce identical output across runs. AEH records configs and seeds, but can't force determinism the backend doesn't offer.
- AEH compares memory architectures. It is not a general agent benchmark. For that, see [Inspect AI](https://inspect.aisi.org.uk/) or AgentBench.

## Roadmap

1. Core foundation (config, data contracts, errors, logging)
2. Inference layer (fake model, llama.cpp adapter)
3. Tasks and architectures
4. Runner, traces, fault and attack scenarios
5. Metrics, reporting, CLI
6. Installers and V1 release

Later: a terminal UI, plus more backends (CUDA, ROCm, SYCL, Apple Silicon).

## Install (once V1 ships)

Supports Ubuntu 22.04+ and Windows 11, Python 3.11+. Minimum 16 GB RAM for CPU inference; any Vulkan GPU helps.

```bash
git clone https://github.com/AnishAchutha05/EVAN-agent-evaluation-harness.git
cd EVAN-agent-evaluation-harness
python3 -m venv .venv && source .venv/bin/activate
pip install -e .
```

## Acknowledgements

Builds on ideas from Fayaz Ali's manuscript *Stateless Architectures for Reliable and Controllable LLM-Based Agents*, and positions itself against Inspect AI (UK AISI) and AgentBench (Liu et al., ICLR 2024).

## License

To be decided.