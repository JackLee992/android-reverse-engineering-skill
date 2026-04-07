# Android Runtime Instrumentation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add optional Frida-based runtime DEX/native tracing and IDA MCP handoff support to the Codex Android reverse engineering skill.

**Architecture:** Keep the current static reverse-engineering workflow intact, then add a separate dynamic-analysis branch driven by shell wrappers and Python host scripts. Use external `ida-pro-mcp` as an optional companion instead of vendoring it.

**Tech Stack:** Bash, Python 3, Frida, ADB, optional IDA Pro MCP, Markdown docs

---

### Task 1: Add design and workflow documentation

**Files:**
- Create: `docs/superpowers/specs/2026-04-07-android-runtime-instrumentation-design.md`
- Create: `docs/superpowers/plans/2026-04-07-android-runtime-instrumentation.md`

- [ ] Write the design and implementation plan documents.
- [ ] Review them for scope, missing files, and contradictions.

### Task 2: Add Frida environment helpers and dependency checks

**Files:**
- Create: `codex-skills/android-reverse-engineering/scripts/_frida_env.sh`
- Create: `codex-skills/android-reverse-engineering/scripts/check-dynamic-deps.sh`

- [ ] Add shell helpers that locate a usable Python interpreter with the `frida` module.
- [ ] Add a dynamic dependency checker for ADB, Frida host tooling, device visibility, and optional IDA MCP.
- [ ] Run the checker and verify the output is clear when prerequisites are missing.

### Task 3: Add host drivers and Frida JavaScript probes

**Files:**
- Create: `codex-skills/android-reverse-engineering/scripts/frida_dexdump_host.py`
- Create: `codex-skills/android-reverse-engineering/scripts/frida_trace_host.py`
- Create: `codex-skills/android-reverse-engineering/scripts/frida_dump_dex.js`
- Create: `codex-skills/android-reverse-engineering/scripts/frida_trace_loads.js`
- Create: `codex-skills/android-reverse-engineering/scripts/frida_trace_jni.js`

- [ ] Add a host script that receives dumped DEX blobs from Frida and writes them locally.
- [ ] Add a host script that records structured JSONL trace output for load/JNI tracing.
- [ ] Add JavaScript probes for DEX loading, `dlopen`/`android_dlopen_ext`, and `RegisterNatives`.
- [ ] Run `python3 -m py_compile` for the host scripts.

### Task 4: Add user-facing wrappers and references

**Files:**
- Create: `codex-skills/android-reverse-engineering/scripts/run-frida-dexdump.sh`
- Create: `codex-skills/android-reverse-engineering/scripts/run-frida-trace-loads.sh`
- Create: `codex-skills/android-reverse-engineering/scripts/run-frida-trace-jni.sh`
- Create: `codex-skills/android-reverse-engineering/references/frida-setup.md`
- Create: `codex-skills/android-reverse-engineering/references/dynamic-dex-unpack.md`
- Create: `codex-skills/android-reverse-engineering/references/native-so-tracing.md`
- Create: `codex-skills/android-reverse-engineering/references/ida-pro-mcp.md`

- [ ] Add simple wrapper entrypoints with `--help`.
- [ ] Write references that explain setup, common commands, and expected outputs.
- [ ] Verify every wrapper returns help text without requiring a working device.

### Task 5: Update the main skill entrypoints

**Files:**
- Modify: `codex-skills/android-reverse-engineering/SKILL.md`
- Modify: `README.md`
- Modify: `CODEX-INSTALL.md`

- [ ] Extend the skill documentation with optional dynamic phases and IDA MCP handoff.
- [ ] Update repository docs so Codex users know the new capabilities and install flow.
- [ ] Re-read the docs to ensure the dynamic flow is clearly optional and does not overwrite the static path.

### Task 6: Verify and sync

**Files:**
- Modify: `codex-skills/android-reverse-engineering/scripts/check-dynamic-deps.sh`
- Modify: `codex-skills/android-reverse-engineering/scripts/decompile.sh` only if regression found

- [ ] Run:
  - `bash codex-skills/android-reverse-engineering/scripts/check-dynamic-deps.sh`
  - `bash codex-skills/android-reverse-engineering/scripts/run-frida-dexdump.sh --help`
  - `bash codex-skills/android-reverse-engineering/scripts/run-frida-trace-loads.sh --help`
  - `bash codex-skills/android-reverse-engineering/scripts/run-frida-trace-jni.sh --help`
  - `python3 -m py_compile codex-skills/android-reverse-engineering/scripts/frida_dexdump_host.py codex-skills/android-reverse-engineering/scripts/frida_trace_host.py`
- [ ] Copy updated files into the installed skill directory.
- [ ] Re-run the same verification commands from the installed location.

### Task 7: Commit and publish

**Files:**
- Modify: repository git history only

- [ ] Commit the new dynamic-analysis integration.
- [ ] Push the fork.
- [ ] Prepare concise user instructions with example commands for:
  - static-only analysis
  - runtime DEX dump
  - `.so` load/JNI tracing
  - IDA MCP handoff
