# SDD-Models Profile System — Exploration Report

**Project**: opencode
**Change**: sdd-models
**Phase**: explore
**Date**: 2026-04-01

---

## Executive Summary

This exploration analyzed the `/sdd-models` profile system from `lightcode-main` (TypeScript/React TUI) for porting to the current `opencode` configuration project at `C:\Users\DELL\.config\opencode`.

**Key Discovery**: The current project is a **configuration directory** (not a source repository). It contains:
- `opencode.json` — Agent configurations with SDD phase model assignments
- `AGENTS.md` — Agent instructions
- `commands/` — Slash command definitions
- `skills/` — SDD skill definitions

**Integration Path**: Since the TypeScript code (TUI dialogs, file I/O, config overlay) is already implemented in the `lightcode-main` runtime, the porting task reduces to creating the **file-based artifacts** needed for the feature to work:

1. `.opencode/sdd-models.jsonc` — Profile storage file
2. `commands/sdd-models.md` — Slash command definition (headless mode fallback)

**Critical Requirement**: Default profiles MUST be **deletable** — remove the `isSddBuiltinProfile()` restriction.

---

## 1. lightcode-main Implementation Analysis

### 1.1 File Structure

| File | Purpose | Status |
|------|---------|--------|
| `.opencode/sdd-models.jsonc` | User-editable profile configuration | **Needs creation** |
| `.opencode/commands/sdd-models.md` | Command description (headless fallback) | **Needs creation** |
| `packages/opencode/src/cli/cmd/tui/util/sdd-models-default.ts` | Default profiles + builtin check | Runtime code |
| `packages/opencode/src/cli/cmd/tui/util/sdd-models-file.ts` | File I/O operations | Runtime code |
| `packages/opencode/src/cli/cmd/tui/component/dialog-sdd-models.tsx` | TUI dialog component | Runtime code |
| `packages/opencode/src/config/config.ts` | Config overlay merge (`applySddModelsOverlay`) | Runtime code |

### 1.2 Data Model

```jsonc
// .opencode/sdd-models.jsonc structure
{
  "active": "balanced",  // Currently active profile
  "profiles": {
    "balanced": {
      "sdd-apply": "mistral/codestral-latest",
      "sdd-archive": "opencode/mimo-v2-omni-free",
      "sdd-init": "opencode/kimi-k2.5"
    },
    "quality": {},
    "economy": {}
  }
}
```

Each profile is a map of `agent-name → model-reference`. An empty profile inherits from:
1. `agent.model` in loaded config
2. Primary session model (fallback)

### 1.3 Default Profiles (RESTRICTED in lightcode-main)

**Location**: `sdd-models-default.ts`

```typescript
export const SDD_BUILTIN_PROFILE_NAMES = ["balanced", "quality", "economy"] as const

export function isSddBuiltinProfile(name: string) {
  return (SDD_BUILTIN_PROFILE_NAMES as readonly string[]).includes(name)
}

export const SDD_MODELS_DEFAULT = `{
  "active": "balanced",
  "profiles": {
    "balanced": {},
    "quality": {},
    "economy": {}
  }
}
`
`
```

**RESTRICTION ENFORCEMENT**:
- `dialog-sdd-models.tsx` lines 88-91: Shows warning if attempting to delete builtin profile
- `sdd-models-file.ts` line 111: Throws error on deletion attempt

### 1.4 Config Overlay Mechanism

**Location**: `config.ts` lines 1344-1394

```typescript
async function applySddModelsOverlay(input: { directory: string; worktree: string; agent: Info["agent"] }) {
  // 1. Search for .opencode/sdd-models.jsonc (upward from directory)
  // 2. Parse JSON/JSONC
  // 3. Resolve active profile (env override OPENCODE_SDD_MODEL_PROFILE > file's active)
  // 4. For each agent in profile: agent[name] = mergeDeep(agent[name] ?? {}, { model })
}
```

### 1.5 Environment Variable Override

| Variable | Purpose |
|----------|---------|
| `OPENCODE_SDD_MODEL_PROFILE` | Override active profile without editing file |
| `OPENCODE_DISABLE_PROJECT_CONFIG` | Disable all project config overlays |

---

## 2. Current Project Analysis

### 2.1 Directory Structure

```
C:\Users\DELL\.config\opencode\
├── AGENTS.md              # Agent instructions (Gentleman persona)
├── ORCHESTRATOR.md.disabled # SDD orchestrator (disabled)
├── opencode.json          # Agent configurations with SDD models
├── commands/              # Slash command definitions
│   ├── sdd-apply.md
│   ├── sdd-archive.md
│   ├── sdd-explore.md
│   ├── sdd-init.md
│   └── ...
├── skills/                # SDD skill definitions
│   ├── sdd-apply/
│   ├── sdd-explore/
│   └── ...
└── openspec/              # SDD artifact storage
    └── changes/
```

### 2.2 Current Agent Configuration (opencode.json)

The file defines SDD agents with their models:

```json
{
  "agent": {
    "sdd-apply": {
      "model": "mistral/codestral-latest",
      "fallback_models": ["mistral/devstral-latest", "opencode-go/glm-5"]
    },
    "sdd-explore": {
      "model": "mistral/mistral-large-latest",
      "fallback_models": ["opencode-go/glm-5", "mistral/mixtral-8x22b-instruct"]
    },
    // ... other sdd-* agents
  }
}
```

### 2.3 Gap Analysis

| Feature | lightcode-main | Current Project | Status |
|---------|----------------|-----------------|--------|
| Agent definitions | `opencode.json` | `opencode.json` | ✅ Already exists |
| SDD skills | `.opencode/skills/` | `skills/` | ✅ Already exists |
| Commands | `.opencode/commands/` | `commands/` | ✅ Already exists |
| Profile storage | `.opencode/sdd-models.jsonc` | ❌ Missing | **Needs creation** |
| `/sdd-models` command | `.opencode/commands/sdd-models.md` | ❌ Missing | **Needs creation** |
| Runtime TUI | TypeScript/React | ✅ In lightcode-main binary | Already implemented |

---

## 3. Porting Requirements

### 3.1 User Requirement

> **"The default profiles MUST be deletable — remove any restriction preventing their deletion."**

This means we need to ensure the runtime code does NOT prevent deletion of `balanced`, `quality`, or `economy` profiles.

### 3.2 Files to Create

| File | Purpose | Location |
|------|---------|----------|
| `sdd-models.jsonc` | Profile storage | `.opencode/sdd-models.jsonc` |
| `sdd-models.md` | Command definition | `commands/sdd-models.md` |

### 3.3 Runtime Code Changes Required

The restriction is in the **runtime code** (not in config files). To make default profiles deletable:

| File | Change Required |
|------|-----------------|
| `sdd-models-default.ts` | Remove `isSddBuiltinProfile()` or return `false` always |
| `dialog-sdd-models.tsx` | Remove lines 89-91 that check `isSddBuiltinProfile` |
| `sdd-models-file.ts` | Remove line 111 that throws on builtin deletion |

**Note**: These changes are in the `lightcode-main` TypeScript code, which is compiled into the opencode binary. The current project (a config directory) cannot modify runtime behavior directly.

**Alternative Solution**: Propose the deletion of the restriction in the upstream `lightcode-main` repository, and document that users should use a version with the fix.

For the current project, we can only create the file artifacts. The runtime restriction must be addressed separately.

---

## 4. Proposed Profile Configuration

### 4.1 Initial `.opencode/sdd-models.jsonc`

Based on the current `opencode.json` agent models:

```jsonc
{
  "active": "balanced",
  "profiles": {
    "balanced": {
      "sdd-apply": "mistral/codestral-latest",
      "sdd-archive": "opencode/mimo-v2-omni-free",
      "sdd-design": "anthropic/claude-opus-4-6",
      "sdd-explore": "mistral/mistral-large-latest",
      "sdd-init": "opencode/kimi-k2.5",
      "sdd-propose": "google/gemini-3.1-pro-preview",
      "sdd-spec": "opencode-go/glm-5",
      "sdd-tasks": "google/gemini-3-flash-preview",
      "sdd-verify": "mistral/codestral-latest"
    },
    "quality": {},
    "economy": {}
  }
}
```

### 4.2 Command Definition `commands/sdd-models.md`

```markdown
---
description: SDD model profiles — in TUI opens .opencode/sdd-models.jsonc; headless fallback is this hint
agent: build
subtask: false
---

In the **TUI**, **/sdd-models** opens an in-app screen: pick the **active profile**, then each **`sdd-*` agent** to choose a model with the same picker as **/models** (favorites, providers, search). The file `.opencode/sdd-models.jsonc` is updated on disk. Optional env **`OPENCODE_SDD_MODEL_PROFILE`** overrides `active` to force a specific profile.

For **headless** usage: edit `.opencode/sdd-models.jsonc` directly. Structure:
- `active`: string — current profile name
- `profiles`: map of profile name → map of agent name → `provider/model` string

Example:
```jsonc
{
  "active": "premium",
  "profiles": {
    "balanced": { "sdd-apply": "opencode/minimax-m2.5-free" },
    "premium": { "sdd-apply": "openai/gpt-5-codex" }
  }
}
```
```

---

## 5. Integration Strategy

### 5.1 Phase 1: Create File Artifacts

**Artifacts to create in this project:**

1. `.opencode/sdd-models.jsonc` — Initialize with profiles matching `opencode.json`
2. `commands/sdd-models.md` — Slash command for headless mode

### 5.2 Phase 2: Runtime Fix (Separate Task)

**Proposed changes to `lightcode-main` source:**

1. In `sdd-models-default.ts`: Remove `isSddBuiltinProfile()` or make it return `false`
2. In `dialog-sdd-models.tsx`: Remove the deletion block for builtin profiles
3. In `sdd-models-file.ts`: Remove the deletion restriction

These changes require modifying the TypeScript source and recompiling the opencode binary.

### 5.3 Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     Config Load Sequence                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Load global config (opencode.json)                         │
│     └─ Agent definitions with model + fallback_models           │
│                                                                 │
│  2. Load project config (.opencode/opencode.json)               │
│     └─ Project-specific overrides                               │
│                                                                 │
│  3. Load SDD profiles (.opencode/sdd-models.jsonc)              │
│     ├─ Resolve active profile (env > file's active > "balanced")│
│     └─ Merge: agent[name].model = profile[agentName]            │
│                                                                 │
│  4. Final agent config = merged result                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Files Affected

### 6.1 New Files

| Path | Purpose |
|------|---------|
| `.opencode/sdd-models.jsonc` | Profile storage |
| `commands/sdd-models.md` | Slash command |

### 6.2 No Changes Required

| Path | Reason |
|------|--------|
| `opencode.json` | Already has `sdd-*` agents with models |
| `AGENTS.md` | Unaffected |
| `skills/sdd-*` | Unaffected |
| `ORCHESTRATOR.md` | Unaffected (currently disabled) |

---

## 7. Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Runtime restriction on builtin profiles | Cannot delete defaults via TUI | Propose upstream fix; document workaround |
| JSONC parsing not available | File read fails | Runtime handles JSONC via `jsonc-parser` library |
| Profile names conflict | Unexpected behavior | Validate uniqueness on creation |
| Last profile deletion | No profiles remain | Keep minimum-1-profile check |

---

## 8. Key Learnings

1. **Current project is config-only** — Not a source repo, only stores agent instructions and config
2. **Runtime code controls deletion** — TUI restriction is in TypeScript, not config files
3. **Profile overlay merges onto agent config** — Model override is additive, doesn't replace fallback_models
4. **Env override has highest priority** — `OPENCODE_SDD_MODEL_PROFILE` > `sdd-models.jsonc.active`
5. **Separation of concerns** — Config files are separate from runtime implementation

---

## 9. Artifacts

| Type | Path |
|------|------|
| Exploration Report | `.opencode/openspec/changes/sdd-models/explore.md` |
| Engram Memory | `sdd/sdd-models/explore` |
| Source Project | `C:\Users\DELL\Downloads\lightcode-main` |
| Target Project | `C:\Users\DELL\.config\opencode` |

---

## 10. Next Steps

1. **Create** `.opencode/sdd-models.jsonc` with initial profiles
2. **Create** `commands/sdd-models.md` command definition
3. **Propose** upstream fix to remove builtin profile deletion restriction
4. **Document** workaround for users who need to delete defaults
5. **Proceed** to `sdd-propose` phase to define the change formally