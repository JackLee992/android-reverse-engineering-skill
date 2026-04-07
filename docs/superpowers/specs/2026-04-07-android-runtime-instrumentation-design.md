# Android Runtime Instrumentation Design

## Goal

Extend the Codex-compatible `android-reverse-engineering` skill with optional dynamic-analysis workflows for:

- runtime DEX dumping with Frida
- native library load tracing
- JNI `RegisterNatives` tracing
- structured handoff into `ida-pro-mcp` for `.so` analysis in IDA Pro

The existing static workflow must remain unchanged for users who only need APK/JAR/AAR decompilation and API extraction.

## Scope

### In scope

- New dynamic-analysis references and workflow guidance in `SKILL.md`
- Host-side dependency checks for Frida, ADB, and optional IDA MCP
- Local wrapper scripts for:
  - runtime DEX dumping
  - `.so` load tracing
  - JNI registration tracing
- Frida JavaScript hooks shipped inside the skill package
- IDA MCP setup and usage guidance for Codex users

### Out of scope

- Bundling or vendoring the `ida-pro-mcp` repository
- Full dynamic debugging orchestration inside IDA
- Device rooting, Frida server deployment, or Magisk setup automation
- General-purpose malware sandboxing or Frida bypass packs

## Architecture

The skill remains a single top-level Android reverse engineering skill with an optional runtime branch.

### Static branch

Unchanged:

1. dependency check
2. decompilation
3. structure analysis
4. call-flow tracing
5. API extraction

### Dynamic branch

New optional phases:

6. dynamic runtime analysis with Frida
7. IDA MCP handoff for native deep-dive work

The dynamic branch uses local shell wrappers to locate a usable Python interpreter with the `frida` module, then launches specialized Python host scripts that load Frida JavaScript instrumentation.

## File plan

### New scripts

- `codex-skills/android-reverse-engineering/scripts/_frida_env.sh`
- `codex-skills/android-reverse-engineering/scripts/check-dynamic-deps.sh`
- `codex-skills/android-reverse-engineering/scripts/run-frida-dexdump.sh`
- `codex-skills/android-reverse-engineering/scripts/run-frida-trace-loads.sh`
- `codex-skills/android-reverse-engineering/scripts/run-frida-trace-jni.sh`
- `codex-skills/android-reverse-engineering/scripts/frida_dexdump_host.py`
- `codex-skills/android-reverse-engineering/scripts/frida_trace_host.py`
- `codex-skills/android-reverse-engineering/scripts/frida_dump_dex.js`
- `codex-skills/android-reverse-engineering/scripts/frida_trace_loads.js`
- `codex-skills/android-reverse-engineering/scripts/frida_trace_jni.js`

### New references

- `codex-skills/android-reverse-engineering/references/frida-setup.md`
- `codex-skills/android-reverse-engineering/references/dynamic-dex-unpack.md`
- `codex-skills/android-reverse-engineering/references/native-so-tracing.md`
- `codex-skills/android-reverse-engineering/references/ida-pro-mcp.md`

### Modified files

- `codex-skills/android-reverse-engineering/SKILL.md`
- `README.md`
- `CODEX-INSTALL.md`

## Key decisions

### 1. Keep `ida-pro-mcp` external

`ida-pro-mcp` is already a mature MCP server with its own install flow. The skill should detect and document it, not vendor it.

### 2. Improve on `frida-unpack` rather than copy it verbatim

The upstream proof-of-concept demonstrates the right hook points, but it hardcodes output behavior and relies on older symbol assumptions. The skill will reuse the core idea while:

- writing DEX files locally on the host
- supporting both spawn and attach modes
- probing common `OpenMemory` and `OpenCommon` hook points
- logging structured events for Codex-friendly analysis

### 3. Prefer wrappers over assuming one Python

Frida environments are often broken because the CLI and the Python module live under different interpreters. The wrappers should resolve a usable interpreter instead of assuming `python3` is correct.

## Verification strategy

- Static verification:
  - `check-dynamic-deps.sh`
  - `run-frida-dexdump.sh --help`
  - `run-frida-trace-loads.sh --help`
  - `run-frida-trace-jni.sh --help`
  - `python3 -m py_compile` on host scripts
- Existing static workflow smoke test:
  - keep the previously working JAR decompilation smoke test passing
- Dynamic runtime execution against a real device is optional and environment-dependent, so the first implementation must fail clearly when Frida/device prerequisites are missing

## Risks

- Frida symbol names vary by Android version and vendor builds
- Host Python/Frida mismatches are common
- JNI tracing coverage varies when apps load or register natives in unusual ways
- IDA MCP integration is only useful when IDA Pro is installed and configured separately

## Mitigations

- Probe multiple libraries and symbol patterns for DEX loading
- Centralize Frida interpreter resolution in one shell helper
- Keep output formats simple: local `.dex` files and JSONL traces
- Make IDA MCP explicitly optional in both docs and dependency checks
