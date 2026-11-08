aeh/
├── pyproject.toml
├── requirements.txt
├── README.md
├── .gitignore
├── .env.example
├── configs/
│   ├── pilot.yaml                       # 5 primary metrics only (report-compliant)
│   ├── full_security.yaml               # primary + 8 security + hallucination
│   └── examples/
│       ├── llama_cpp_local.yaml
│       └── custom_architecture.yaml
├── benchmarks/
│   └── pilot_v1/
│       ├── tasks.json                   # 5-8 step tasks + ground truth
│       ├── faults.json                  # fault scenarios (EPI)
│       ├── injections.json              # prompt injections (AMR, IPR, ADR)
│       ├── security_probes.json         # role override, exfil, scope, command, payload
│       ├── tool_policies.json           # allowed/forbidden tools per task (TBR, SCR)
│       └── reference_facts.json         # grounding facts (Hallucination Rate)
├── docs/
│   ├── hld.md
│   ├── lld.md
│   ├── cli_design.md
│   ├── metrics_spec.md                  # all 14: formula, denominator, missing-data rule
│   ├── methodology.md
│   ├── custom_architectures.md
│   ├── installation.md
│   └── decisions/
│       ├── 0001-metric-tiers.md
│       └── 0002-tool-sandbox.md
├── scripts/
│   ├── install_ubuntu.sh
│   ├── install_windows.ps1
│   ├── setup_llamacpp.sh
│   ├── check_vulkan.sh
│   └── validate_install.sh
├── results/
│   └── .gitkeep
├── src/aeh/
│   ├── __init__.py
│   ├── __main__.py
│   ├── py.typed
│   ├── core/                            # LAYER 1
│   │   ├── config.py                    # YAML/TOML loading
│   │   ├── schema.py                    # config models + validation
│   │   ├── models.py                    # Task, Step, Cycle, RunResult
│   │   ├── errors.py
│   │   ├── logging.py                   # JSON logs + secret redaction
│   │   ├── registry.py                  # generic Registry[T]
│   │   └── ids.py                       # run IDs, seeds
│   ├── providers/                       # LAYER 2
│   │   ├── base.py                      # ModelProvider ABC
│   │   ├── fake.py
│   │   ├── llama_cpp.py
│   │   ├── normalize.py
│   │   ├── retry.py
│   │   ├── health.py
│   │   └── factory.py
│   ├── tools/                           # LAYER 3 (sandboxed mock tools)
│   │   ├── base.py                      # Tool ABC
│   │   ├── sandbox.py                   # no real network/fs/exec
│   │   ├── policy.py                    # allowed/forbidden tool policy
│   │   └── builtin.py                   # mock calculator, lookup, file, shell
│   ├── tasks/                           # LAYER 3
│   │   ├── base.py
│   │   ├── loader.py
│   │   ├── checker.py                   # ground-truth success
│   │   └── equivalence.py               # output equivalence (OCS)
│   ├── architectures/                   # LAYER 3
│   │   ├── base.py                      # AgentBase ABC
│   │   ├── stateful.py
│   │   ├── ephemeral.py
│   │   ├── hybrid.py
│   │   ├── operator.py
│   │   ├── memory_validator.py
│   │   └── registry.py
│   ├── scenarios/                       # LAYER 4
│   │   ├── base.py
│   │   ├── fault_injection.py
│   │   ├── prompt_injection.py
│   │   └── security/
│   │       ├── role_override.py         # IPR, SRR
│   │       ├── tool_abuse.py            # TBR, UCR
│   │       ├── exfiltration.py          # EER (canary tokens)
│   │       ├── scope_creep.py           # SCR
│   │       └── payload.py               # MPC
│   ├── trace/                           # LAYER 4
│   │   ├── models.py
│   │   ├── logger.py
│   │   └── store.py
│   ├── runner/                          # LAYER 4
│   │   ├── experiment.py
│   │   ├── scheduler.py
│   │   ├── executor.py
│   │   ├── counterfactual.py            # EPI replay
│   │   ├── attribution.py
│   │   └── recovery.py
│   ├── detectors/                       # LAYER 4/5 (shared by security metrics)
│   │   ├── base.py                      # Detector ABC
│   │   ├── canary.py                    # planted secret leak detection
│   │   ├── tool_policy.py               # forbidden tool call detection
│   │   ├── command_scan.py              # unintended command detection
│   │   ├── payload_scan.py              # malicious payload execution/echo
│   │   ├── role_check.py                # system-role retention
│   │   ├── claim_check.py               # unsupported claims (hallucination)
│   │   └── attack_signal.py             # agent flagged the attack (ADR)
│   ├── metrics/                         # LAYER 5
│   │   ├── base.py                      # Metric ABC (tier, name, compute)
│   │   ├── registry.py
│   │   ├── tiers.py                     # PRIMARY / SECURITY / QUALITY enums
│   │   ├── aggregation.py
│   │   ├── primary/                     # exactly five
│   │   │   ├── tsr.py                   # Task Success Rate
│   │   │   ├── epi.py                   # Error Propagation Index
│   │   │   ├── ocs.py                   # Output Consistency Score
│   │   │   ├── amr.py                   # Adversarial Manipulation Rate
│   │   │   └── il.py                    # Inference Latency
│   │   ├── security/                    # opt-in, 8 metrics
│   │   │   ├── ipr.py                   # Instruction Priority Resistance
│   │   │   ├── srr.py                   # System Role Retention Rate
│   │   │   ├── tbr.py                   # Tool Boundary Respect
│   │   │   ├── eer.py                   # Exfiltration Evasion Rate
│   │   │   ├── scr.py                   # Scope Containment Rate
│   │   │   ├── ucr.py                   # Unintended Command Rate
│   │   │   ├── mpc.py                   # Malicious Payload Containment
│   │   │   └── adr.py                   # Attack Detection Rate
│   │   ├── quality/                     # opt-in
│   │   │   └── hallucination.py         # Hallucination Rate
│   │   └── diagnostics/                 # NOT metrics
│   │       └── fault_propagation_depth.py
│   ├── reporting/                       # LAYER 5
│   │   ├── table.py                     # tiered Rich tables
│   │   ├── csv_export.py
│   │   └── plots.py
│   └── cli/                             # LAYER 5
│       ├── app.py                       # Typer root, command groups
│       ├── options.py                   # shared option definitions
│       ├── output.py                    # Rich console helpers, --json mode
│       ├── exit_codes.py
│       └── commands/
│           ├── init.py
│           ├── validate.py
│           ├── doctor.py
│           ├── run.py
│           ├── resume.py
│           ├── replay.py
│           ├── models.py                # models check
│           ├── arch.py                  # arch list / info
│           ├── metrics.py               # metrics list / info
│           ├── benchmark.py             # benchmark list / inspect
│           ├── runs.py                  # runs list / show / delete
│           ├── report.py                # report show / compare
│           ├── export.py                # export csv
│           └── plot.py
└── tests/
    ├── conftest.py
    ├── fixtures/
    │   ├── sample_config.yaml
    │   ├── invalid_config.yaml
    │   └── sample_trace.json
    ├── unit/
    │   ├── core/          test_schema, test_models, test_config, test_errors, test_logging
    │   ├── providers/     test_fake, test_llama_cpp, test_retry
    │   ├── tools/         test_sandbox, test_policy
    │   ├── architectures/ test_stateful, test_ephemeral, test_hybrid,
    │   │                  test_memory_validator, test_custom_registration
    │   ├── scenarios/     test_fault_injection, test_prompt_injection, test_security_scenarios
    │   ├── detectors/     test_canary, test_tool_policy, test_command_scan,
    │   │                  test_payload_scan, test_role_check, test_claim_check, test_attack_signal
    │   ├── runner/        test_executor, test_counterfactual, test_attribution, test_recovery
    │   ├── metrics/       test_tsr, test_epi, test_ocs, test_amr, test_il,
    │   │                  test_ipr, test_srr, test_tbr, test_eer, test_scr,
    │   │                  test_ucr, test_mpc, test_adr, test_hallucination,
    │   │                  test_fpd, test_aggregation, test_registry_tiers
    │   └── cli/           test_commands, test_exit_codes
    └── integration/
        ├── test_pipeline_fake_model.py
        ├── test_security_pipeline_fake_model.py
        └── test_llama_cpp_live.py       # skipped unless server running
