# SDD-Models Integration — Exploration Report

**Project**: opencode
**Change**: sdd-models-integration
**Phase**: explore
**Date**: 2026-04-01

---

## Executive Summary

This exploration analyzed the `/sdd-models` feature from `lightcode-main` (a TypeScript fork of opencode) and assessed integration feasibility into the `gentle-ai` project located at `C:\Users\DELL\gentle-ai\`.

**Critical Discovery**: The target repository is a **Go + Bubbletea TUI implementation**, not TypeScript. This requires porting the `/sdd-models` feature concepts to Go rather than copying TypeScript code.

**Integration Path**: The gentle-ai project already has the infrastructure for model selection (`ModelAssignments` type, `model_picker.go` TUI screen), but lacks **profile persistence** and **named profile management**. The feature gap can be filled by implementing:
1. Go types for SDD profiles (`internal/model/sdd_profiles.go`)
2. File I/O for profile storage (`internal/config/sdd_profiles_file.go`)
3. TUI screen for profile management (`internal/tui/screens/sdd_profiles.go`)

---

## 1. lightcode-main Implementation Analysis

### 1.1 File Structure

| File | Purpose |
|------|---------|
| `.opencode/sdd-models.jsonc` | User-editable profile configuration |
| `.opencode/commands/sdd-models.md` | Command description (headless fallback) |
| `packages/opencode/src/cli/cmd/tui/util/sdd-models-default.ts` | Default profiles + builtin check |
| `packages/opencode/src/cli/cmd/tui/util/sdd-models-file.ts` | File I/O operations |
| `packages/opencode/src/cli/cmd/tui/component/dialog-sdd-models.tsx` | TUI dialog component |
| `packages/opencode/src/config/config.ts` | Config overlay merge function |
| `docs/sdd-models-tui-guide.md` | User documentation |

### 1.2 Data Model

```typescript
// sdd-models.jsonc structure
{
  "active": "premium",  // Current active profile
  "profiles": {
    "balanced": {
      "sdd-apply": "opencode/minimax-m2.5-free",
      "sdd-archive": "opencode/minimax-m2.5-free",
      "sdd-init": "opencode/minimax-m2.5-free"
    },
    "quality": {},
    "economy": {},
    "premium": {
      "sdd-apply": "openai/gpt-5-codex",
      "sdd-archive": "opencode/minimax-m2.5-free",
      "sdd-init": "opencode/nemotron-3-super-free"
    }
  }
}
```

### 1.3 Default Profiles (RESTRICTED)

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
```

### 1.4 Restriction Points

The restriction preventing deletion of default profiles is enforced in **two places**:

#### A. TUI Dialog (`dialog-sdd-models.tsx`, lines 88-91)

```typescript
if (isSddBuiltinProfile(name)) {
  toast.show({ message: `Built-in profile "${name}" cannot be deleted`, variant: "warning" })
  return
}
```

#### B. File Operations (`sdd-models-file.ts`, line 110-111)

```typescript
export async function deleteSddProfile(filepath: string, name: string) {
  if (isSddBuiltinProfile(name)) throw new Error(`Built-in profile "${name}" cannot be deleted`)
  // ... rest of deletion logic
}
```

### 1.5 Config Overlay Mechanism

**Location**: `packages/opencode/src/config/config.ts`, lines 1344-1394

The `applySddModelsOverlay()` function:
1. Searches for `.opencode/sdd-models.jsonc` or `.opencode/sdd-models.json` (upward from working directory)
2. Parses and validates the file
3. Resolves active profile (env override `OPENCODE_SDD_MODEL_PROFILE` > file's `active`)
4. Merges profile mappings onto agent configs: `agent[name] = mergeDeep(agent[name] ?? {}, { model })`

### 1.6 TUI Command Registration

**Location**: `app.tsx`, lines 511-521

```typescript
{
  title: "SDD model profiles",
  value: "sdd.models",
  category: "Agent",
  slash: { name: "sdd-models" },
  onSelect: () => {
    dialog.replace(() => <DialogSddModels />)
  },
}
```

### 1.7 Autocomplete Deduplication

**Location**: `autocomplete.tsx`, lines 360-384

The system prevents duplicate `/sdd-models` entries by:
1. Collecting app-registered slashes: `command.slashes()`
2. Building a set of names: `const appSlashNames = new Set(...)`
3. Skipping server commands that match: `if (appSlashNames.has(serverCommand.name)) continue`

---

## 2. gentle-ai Target Project Analysis

### 2.1 Target Repository Location

**Path**: `C:\Users\DELL\gentle-ai\`

This is the **source repository** for the gentle-ai/opencode CLI, written in **Go** with a **Bubbletea TUI** (github.com/charmbracelet/bubbletea).

### 2.2 Key Architecture Difference

| Aspect | lightcode-main | gentle-ai |
|--------|----------------|-----------|
| Language | TypeScript | Go |
| TUI Framework | React/Ink | Bubbletea |
| Config File | sdd-models.jsonc | Not implemented yet |
| Profile Management | TypeScript classes | Need to implement in Go |
| Overlay Mechanism | config.ts | internal/components/sdd/inject.go |

### 2.3 Directory Structure

```
C:\Users\DELL\gentle-ai\
├── cmd/                              # CLI entry points
│   └── gentle-ai/                    # Main binary
├── internal/
│   ├── model/                        # Domain types
│   │   ├── types.go                  # SDDModeID, AgentID, etc.
│   │   ├── model_pool.go              # ModelPool, ModelAssignments types
│   │   ├── model_assignment.go        # Model assignment logic
│   │   └── selection.go               # Selection struct (ModelAssignments field)
│   ├── tui/                          # Bubbletea TUI
│   │   ├── model.go                  # Main TUI model
│   │   ├── router.go                 # Screen routing
│   │   └── screens/
│   │       ├── model_picker.go       # Model selection per phase
│   │       ├── sdd_mode.go           # Single/Multi-agent mode selection
│   │       └── ...                   # Other screens
│   ├── components/
│   │   └── sdd/
│   │       └── inject.go             # SDD configuration injection
│   ├── cli/                          # CLI commands
│   └── opencode/                     # OpenCode-specific logic
└── skills/                           # Skill definitions
```

### 2.4 Current Implementation Status

**Already Implemented:**
- `model.ModelAssignments` type (internal/model/model_pool.go)
- `Selection.ModelAssignments` field for in-memory assignments
- `model_picker.go` - TUI screen for selecting models per SDD phase
- `sdd_mode.go` - TUI screen for single/multi-agent mode
- `sdd.Inject()` function for overlaying model assignments onto agent configs

**NOT Implemented (gap to fill):**
- Profile persistence (saving/loading model assignments to a file)
- Named profiles (balanced, quality, economy, etc.)
- `/sdd-models` command or TUI screen for profile management
- JSONC/JSON file for storing profiles

### 2.5 Model Pool Structure

```go
// internal/model/model_pool.go
type ModelPool struct {
    Primary   ModelReference   // First model
    Fallbacks []ModelReference // Backup models in order
}

type ModelAssignments map[string]ModelPool
// Key = phase name (e.g., "sdd-propose", "sdd-apply")
```

---

## 3. Integration Requirements

### 3.1 User Requirement

> "The default profiles should be kept, but the user must be able to delete them without any restrictions."

### 3.2 Technical Requirements

To implement `/sdd-models` with deletable default profiles:

| Requirement | Location | Change |
|-------------|----------|--------|
| Remove deletion restriction | `sdd-models-default.ts` | Remove `isSddBuiltinProfile()` or return `false` |
| Remove TUI restriction | `dialog-sdd-models.tsx` | Remove the `isSddBuiltinProfile` check (lines 88-91) |
| Remove file restriction | `sdd-models-file.ts` | Remove line 111 restriction |
| Add `/sdd-models` command | `commands/sdd-models.md` | Create command file |
| Add TUI dialog | `dialog-sdd-models.tsx` | Port TUI component |
| Add file utilities | `sdd-models-file.ts` | Port file operations |
| Add config overlay | `config.ts` | Add `applySddModelsOverlay()` function |
| Register command in TUI | `app.tsx` | Add slash command registration |
| Add autocomplete dedup | `autocomplete.tsx` | Ensure no duplicate entries |

---

## 4. Integration Strategy

### 4.1 Target Repository Confirmed

**Path**: `C:\Users\DELL\gentle-ai\`
**Language**: Go
**TUI Framework**: Bubbletea

### 4.2 Recommended Approach

Port the `/sdd-models` feature from TypeScript to Go:

1. **Phase 1: Core Infrastructure (Go)**
   - Create `internal/model/sdd_profiles.go` with profile types and defaults
   - Create `internal/config/sdd_profiles_file.go` for file I/O (JSON/JSONC)
   - Add profile loading to `internal/components/sdd/inject.go`

2. **Phase 2: TUI Integration (Bubbletea)**
   - Create `internal/tui/screens/sdd_profiles.go` for profile management
   - Add profile selection to the existing model picker flow
   - Integrate with `internal/tui/model.go` router

3. **Phase 3: Command Integration**
   - Create `internal/opencode/sdd_profiles.go` for profile discovery
   - Add `/sdd-models` slash command support

### 4.3 File Mapping (TypeScript → Go)

| lightcode-main (TS) | gentle-ai (Go) |
|---------------------|----------------|
| `sdd-models-default.ts` | `internal/model/sdd_profiles.go` |
| `sdd-models-file.ts` | `internal/config/sdd_profiles_file.go` |
| `dialog-sdd-models.tsx` | `internal/tui/screens/sdd_profiles.go` |
| `config.ts` (overlay) | `internal/components/sdd/inject.go` |
| `app.tsx` (command reg) | `internal/tui/router.go` |

### 4.4 Data Model (Go)

```go
// internal/model/sdd_profiles.go

type SDDProfile struct {
    Name    string            `json:"name"`
    Models  map[string]string `json:"models"` // phase -> model reference
}

type SDDProfilesConfig struct {
    Active   string       `json:"active"`
    Profiles []SDDProfile `json:"profiles"`
}

// Default profiles - all deletable by user
var DefaultProfiles = []SDDProfile{
    {Name: "balanced", Models: map[string]string{
        "sdd-apply": "opencode/minimax-m2.5-free",
        // ...
    }},
    {Name: "quality", Models: map[string]string{}},
    {Name: "economy", Models: map[string]string{}},
}
```

### 4.5 Key Behavioral Change

**Original (lightcode-main)**: Default profiles cannot be deleted (`isSddBuiltinProfile()` check)
**New (gentle-ai)**: No deletion restrictions - all profiles are editable/deletable

```go
// No builtin profile restriction - all profiles are user-editable
// Validation only ensures at least one profile remains
func (c *SDDProfilesConfig) CanDelete(name string) error {
    if len(c.Profiles) <= 1 {
        return errors.New("cannot delete the last profile")
    }
    return nil
}
```

---

## 5. Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Last profile deletion | User could delete all profiles | Add check: minimum 1 profile required |
| JSON/JSONC parsing | Invalid file breaks overlay | Robust error handling with fallback to defaults |
| Go type conversion | TypeScript patterns differ | Re-design data structures for Go idioms |
| Go module conflicts | Bubbletea version mismatches | Use existing module versions from go.mod |
| File location | Need to decide where to store profiles | Use `.opencode/sdd-profiles.json` alongside opencode.json |

---

## 6. Artifacts

| Artifact | Path |
|----------|------|
| Exploration Report | `C:\Users\DELL\.config\opencode\openspec\changes\sdd-models-integration\explore.md` |
| Engram Memory | `sdd/sdd-models-integration/explore` |
| Target Repository | `C:\Users\DELL\gentle-ai\` |
| TUI Source | `C:\Users\DELL\gentle-ai\internal\tui\` |
| Model Types | `C:\Users\DELL\gentle-ai\internal\model\` |

---

## 7. Next Steps

1. **Design**: Create Go type definitions for SDD profiles
2. **Implement**: File I/O for `sdd-profiles.json` in `internal/config/`
3. **Implement**: Default profiles in `internal/model/sdd_profiles.go`
4. **Implement**: TUI screen `internal/tui/screens/sdd_profiles.go`
5. **Integrate**: Hook profiles into model picker flow
6. **Test**: Profile CRUD operations and model injection

---

## 8. Key Learnings

1. **gentle-ai is Go + Bubbletea** — not TypeScript, requires porting concepts
2. **Already has ModelAssignments** — in-memory structure exists, needs persistence
3. **Model picker exists** — TUI for selecting models per phase already implemented
4. **No profile system yet** — gap to fill with this feature
5. **Inject mechanism exists** — `sdd.Inject()` handles config overlay