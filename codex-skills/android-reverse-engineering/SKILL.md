---
name: android-reverse-engineering
description: Use when the user wants to decompile an Android APK, XAPK, JAR, or AAR, inspect obfuscated Android code, extract HTTP APIs such as Retrofit or OkHttp endpoints, or trace call flows from UI code down to the network layer without access to the original source.
---

# Android Reverse Engineering

Decompile Android APK, XAPK, JAR, and AAR files using jadx and Fernflower/Vineflower, trace call flows through application code and libraries, and produce structured documentation of extracted APIs. Two decompiler engines are supported: jadx for broad Android coverage and Fernflower/Vineflower for higher-quality output on complex Java code. They can also be used together for comparison.

## Installation Path

This Codex packaging assumes the skill is installed at:

```bash
$HOME/.codex/skills/android-reverse-engineering
```

That is the default destination used by the bundled Codex skill installer. Use the commands below as written unless you intentionally installed the skill somewhere else.

## Prerequisites

This skill requires **Java JDK 17+** and **jadx** to be installed. **Fernflower/Vineflower** and **dex2jar** are optional but recommended for better decompilation quality. Run the dependency checker to verify:

```bash
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/check-deps.sh"
```

If anything is missing, follow the installation instructions in:

```text
$HOME/.codex/skills/android-reverse-engineering/references/setup-guide.md
```

## Workflow

### Phase 1: Verify and Install Dependencies

Before decompiling, confirm that the required tools are available and install any that are missing.

**Action**: Run the dependency check script.

```bash
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/check-deps.sh"
```

The output contains machine-readable lines:
- `INSTALL_REQUIRED:<dep>`: must be installed before proceeding
- `INSTALL_OPTIONAL:<dep>`: recommended but not blocking

**If required dependencies are missing** (exit code 1), install them automatically:

```bash
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/install-dep.sh" <dep>
```

The install script detects the OS and package manager, then:
- Installs without sudo when possible by downloading to `~/.local/share/` and symlinking in `~/.local/bin/`
- Uses sudo and the system package manager when necessary (`apt`, `dnf`, `pacman`)
- If sudo is needed but unavailable or declined, prints the exact manual command and exits with code 2. Show those instructions to the user.

**For optional dependencies**, ask the user if they want to install them. Vineflower and dex2jar are recommended for best results.

After installation, re-run `check-deps.sh` to confirm everything is in place. Do not proceed to Phase 2 until all required dependencies are OK.

### Phase 2: Decompile

Use the decompile wrapper script to process the target file. The script supports three engines: `jadx`, `fernflower`, and `both`.

**Action**: Choose the engine and run the decompile script. The script handles APK, XAPK, JAR, and AAR files.

```bash
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/decompile.sh" [OPTIONS] <file>
```

For **XAPK** files, the script automatically extracts the archive, identifies all APK files inside, and decompiles each one into a separate subdirectory. The XAPK manifest is copied to the output for reference.

Options:
- `-o <dir>`: custom output directory (default: `<filename>-decompiled`)
- `--deobf`: enable deobfuscation, recommended for obfuscated apps
- `--no-res`: skip resources and decompile code only
- `--engine ENGINE`: `jadx` (default), `fernflower`, or `both`

**Engine selection strategy**:

| Situation | Engine |
|---|---|
| First pass on any APK | `jadx` |
| JAR/AAR library analysis | `fernflower` |
| jadx output has warnings or broken code | `both` |
| Complex lambdas, generics, or streams | `fernflower` |
| Quick overview of a large APK | `jadx --no-res` |

When using `--engine both`, the outputs go into `<output>/jadx/` and `<output>/fernflower/`, with a comparison summary at the end showing file counts and jadx warning counts. Review classes with jadx warnings in the Fernflower output for better code.

For APK files with Fernflower, the script automatically uses dex2jar as an intermediate step. dex2jar must be installed for this to work.

See these local references for more detail:

```text
$HOME/.codex/skills/android-reverse-engineering/references/jadx-usage.md
$HOME/.codex/skills/android-reverse-engineering/references/fernflower-usage.md
```

### Phase 3: Analyze Structure

Navigate the decompiled output to understand the app's architecture.

**Actions**:

1. Read `AndroidManifest.xml` from `<output>/resources/AndroidManifest.xml`:
   - Identify the main launcher Activity
   - List all Activities, Services, BroadcastReceivers, and ContentProviders
   - Note permissions, especially `INTERNET` and `ACCESS_NETWORK_STATE`
   - Find the application class from `android:name` on `<application>`

2. Survey the package structure under `<output>/sources/`:
   - Identify the main app package and sub-packages
   - Distinguish app code from third-party libraries
   - Look for packages named `api`, `network`, `data`, `repository`, `service`, `retrofit`, or `http`

3. Identify the architecture pattern:
   - MVP: look for `Presenter` classes
   - MVVM: look for `ViewModel` classes and `LiveData` or `StateFlow`
   - Clean Architecture: look for `domain`, `data`, and `presentation` packages

### Phase 4: Trace Call Flows

Follow execution paths from user-facing entry points down to network calls.

**Actions**:

1. Start from entry points: read the main Activity or Application class identified in Phase 3.
2. Follow the initialization chain: `Application.onCreate()` often sets up the HTTP client, base URL, and dependency injection.
3. Trace user actions:
   - `onCreate()` -> view setup -> click listeners
   - click handler -> ViewModel or Presenter method
   - ViewModel -> Repository -> API service interface
   - API service -> actual HTTP call
4. Map DI bindings if Dagger or Hilt is used by finding `@Module` classes.
5. Handle obfuscated code by using string literals and library API calls as anchors. Retrofit annotations and URL strings are typically not obfuscated.

See:

```text
$HOME/.codex/skills/android-reverse-engineering/references/call-flow-analysis.md
```

### Phase 5: Extract and Document APIs

Find all API endpoints and produce structured documentation.

**Action**: Run the API search script for a broad sweep.

```bash
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/find-api-calls.sh" <output>/sources/
```

Targeted searches:

```bash
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/find-api-calls.sh" <output>/sources/ --retrofit
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/find-api-calls.sh" <output>/sources/ --urls
bash "$HOME/.codex/skills/android-reverse-engineering/scripts/find-api-calls.sh" <output>/sources/ --auth
```

Then, for each discovered endpoint, read the surrounding source code to extract:
- HTTP method and path
- Base URL
- Path parameters, query parameters, and request body
- Headers, especially authentication
- Response type
- Where it is called from in the Phase 4 call chain

Document each endpoint using this format:

```markdown
### `METHOD /path`

- **Source**: `com.example.api.ApiService` (`ApiService.java:42`)
- **Base URL**: `https://api.example.com/v1`
- **Path params**: `id` (`String`)
- **Query params**: `page` (`int`), `limit` (`int`)
- **Headers**: `Authorization: Bearer <token>`
- **Request body**: `{ "email": "string", "password": "string" }`
- **Response**: `ApiResponse<User>`
- **Called from**: `LoginActivity -> LoginViewModel -> UserRepository -> ApiService`
```

See:

```text
$HOME/.codex/skills/android-reverse-engineering/references/api-extraction-patterns.md
```

## Output

At the end of the workflow, deliver:

1. Decompiled source in the output directory
2. An architecture summary covering app structure, main packages, and pattern used
3. API documentation for all discovered endpoints
4. A call-flow map for key paths from UI to network, especially authentication and main features

## References

- `$HOME/.codex/skills/android-reverse-engineering/references/setup-guide.md`
- `$HOME/.codex/skills/android-reverse-engineering/references/jadx-usage.md`
- `$HOME/.codex/skills/android-reverse-engineering/references/fernflower-usage.md`
- `$HOME/.codex/skills/android-reverse-engineering/references/api-extraction-patterns.md`
- `$HOME/.codex/skills/android-reverse-engineering/references/call-flow-analysis.md`
