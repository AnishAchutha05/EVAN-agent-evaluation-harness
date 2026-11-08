AEH CLI DESIGN
==============
Binary: aeh        (also: python -m aeh)
Global flags:  --config/-c PATH   --json   --quiet/-q   --verbose/-v   --no-color   --version   --help

COMMAND TREE
------------
aeh init [--dir .] [--preset pilot|full_security]      scaffold configs/ + .env.example
aeh validate -c CONFIG                                  schema + benchmark + architecture checks
aeh doctor                                              python, deps, llama-server, vulkan, writable results/

aeh models check -c CONFIG                              ping server, test completion, show latency
aeh arch list                                           stateful, ephemeral, hybrid + custom
aeh arch info NAME                                      state policy, memory behaviour, fairness notes
aeh metrics list [--tier primary|security|quality|all]  name, tier, short definition
aeh metrics info NAME                                   formula, denominator, missing-data rule
aeh benchmark list
aeh benchmark inspect NAME [--task ID]

aeh run -c CONFIG [options]                             execute experiment
aeh resume RUN_ID                                       continue incomplete run
aeh replay RUN_ID | -c SAVED_CONFIG                     rerun from saved config + seed

aeh runs list [--limit N]
aeh runs show RUN_ID
aeh runs delete RUN_ID [--yes]

aeh report show RUN_ID [--tier primary|security|quality|all]
aeh report compare RUN_ID_A RUN_ID_B
aeh export csv RUN_ID [--out PATH] [--tier all]
aeh plot RUN_ID [--kind fpd|bars|latency] [--out PATH]

RUN OPTIONS
-----------
--arch/-a NAME            repeatable, max 3 (stateful, ephemeral, hybrid, custom:module.Class)
--model-provider TEXT     llama_cpp (default) | custom
--base-url URL            default http://127.0.0.1:8080/v1
--model-id TEXT
--benchmark NAME          default pilot_v1
--tasks N                 default 20
--runs N                  default 3
--temperature FLOAT       default 0.0
--seed INT                default 42
--max-tokens INT          default 512
--timeout SECONDS         default 60
--retries INT             default 2
--inject-step INT         default 2
--faults / --no-faults    default on
--injections / --no-injections   default on
--metrics TEXT            primary (default) | security | quality | all | comma list e.g. tsr,epi,ipr
--out DIR                 default results/
--dry-run                 validate + print plan, no inference
--fake-model              use fake provider (no server needed)

EXAMPLES
--------
aeh doctor
aeh run -c configs/pilot.yaml --fake-model
aeh run -c configs/pilot.yaml -a stateful -a ephemeral -a hybrid
aeh run -c configs/full_security.yaml --metrics all
aeh report show 20261108-231500-a1b2 --tier security
aeh export csv 20261108-231500-a1b2 --out out.csv

OUTPUT
------
Default: Rich tables.  --json: machine-readable on stdout, logs on stderr.
Report tables are tiered:
  Table 1  PRIMARY (TSR EPI OCS AMR IL)       always shown
  Table 2  SECURITY (IPR SRR TBR EER SCR UCR MPC ADR)   if selected
  Table 3  QUALITY (Hallucination Rate)       if selected
  Footer   FPD diagnostic note, run ID, config hash, seed, "preliminary, not statistically conclusive"

EXIT CODES
----------
0 success   1 general error   2 invalid config/usage   3 model unreachable
4 partial run (some runs failed, results saved)   5 interrupted (resumable)

SECURITY RULES
--------------
API keys only from env vars (.env), never printed, never in traces/CSV.
Injected content never executed; all tools run in sandbox.
