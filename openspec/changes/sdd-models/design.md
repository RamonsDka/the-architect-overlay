# Design: sdd-models Profile System Port

## Technical Approach

Port the `/sdd-models` profile system from `lightcode-main` to `C:\Users\DELL\.config\opencode` by:

1. **Creating two config artifacts** that the runtime already knows how to consume.
2. **Patching three runtime TypeScript files** in `lightcode-main` to remove the builtin-profile deletion restriction (`isSddBuiltinProfile` guard).

The runtime overlay pipeline (`config.ts → applySddModelsOverlay`) is already compiled in the opencode binary. No new runtime code is needed for the config side. The deletion restriction is purely a TUI-layer guard — removing it requires source changes in `lightcode-main`.

---

## Architecture Decisions

| # | Decision | Choice | Rejected | Rationale |
|---|----------|--------|----------|-----------|
| 1 | Config artifact location | `.opencode/sdd-models.jsonc` at project root | Global `~/.config/opencode/sdd-models.jsonc` | Runtime `Filesystem.findUp` searches upward from CWD — project-local file takes precedence correctly |
| 2 | Remove builtin restriction | Delete the `isSddBuiltinProfile` guard entirely from all three call sites | Make it return `false` always | Deletion is cleaner; removes dead code rather than leaving a no-op function |
| 3 | Initial profile data | Populate `balanced` with real models from `opencode.json`; leave `quality`/`economy` empty | Copy lightcode-main's minimax entries | The local `opencode.json` already has curated models per agent — seeding from it gives immediate value |
| 4 | CLI command definition | Thin `commands/sdd-models.md` delegating to TUI | Full headless subcommand implementation | The TUI dialog already handles CRUD; the command file is the hook-point that opens it |

---

## Data Flow

```
User types /sdd-models
       │
       ▼
commands/sdd-models.md  ──►  TUI: DialogSddModels
                                    │
                          ensureSddModels(filepath)
                                    │
                    .opencode/sdd-models.jsonc  ◄──  OPENCODE_SDD_MODEL_PROFILE (env override)
                                    │
                         readSddModels() → SddModelsData
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
              profile CRUD                  saveSddModels*()
           (add/delete/switch/set)          atomic file write

──── Config Load Path ────────────────────────────────────
opencode.json (agent[].model)
       +
.opencode/sdd-models.jsonc (active profile overrides)
       │
       ▼
applySddModelsOverlay()
  env OPENCODE_SDD_MODEL_PROFILE > file.active > "default"
  → mergeDeep(agent[name], { model: profile[name] })
       │
       ▼
Final resolved agent config used by SDD phase runners
```

---

## File Changes

### Config artifacts (this project — `C:\Users\DELL\.config\opencode`)

| File | Action | Description |
|------|--------|-------------|
| `.opencode/sdd-models.jsonc` | **Create** | Profile storage file — seeds `balanced` with models from `opencode.json`; `quality`/`economy` empty |
| `commands/sdd-models.md` | **Create** | Slash command hook — opens TUI dialog, documents env override and headless editing |

### Runtime source (lightcode-main — `packages/opencode/src/`)

| File | Action | Description |
|------|--------|-------------|
| `cli/cmd/tui/util/sdd-models-default.ts` | **Modify** | Remove `SDD_BUILTIN_PROFILE_NAMES` const and `isSddBuiltinProfile()` function entirely |
| `cli/cmd/tui/util/sdd-models-file.ts` | **Modify** | Remove line 111: `if (isSddBuiltinProfile(name)) throw new Error(...)` and its import |
| `cli/cmd/tui/component/dialog-sdd-models.tsx` | **Modify** | Remove lines 89–91 (the `isSddBuiltinProfile` toast guard) and its import |

---

## Interfaces / Contracts

```typescript
// sdd-models-file.ts — unchanged public API (no signature changes needed)
type SddModelsData = {
  active: string
  profiles: Record<string, Record<string, string>>
  //        profileName → agentName → "provider/model"
}

// sdd-models-default.ts — after patch (builtin array removed)
export const SDD_MODELS_DEFAULT = `{
  "active": "balanced",
  "profiles": {
    "balanced": {},
    "quality": {},
    "economy": {}
  }
}\n`
// isSddBuiltinProfile() — DELETED

// deleteSddProfile() after patch — line 111 removed:
export async function deleteSddProfile(filepath: string, name: string) {
  // NO builtin guard here anymore
  const d = await readSddModels(filepath)
  if (!d.profiles[name]) throw new Error(`Profile "${name}" not found`)
  const keys = Object.keys(d.profiles)
  if (keys.length <= 1) throw new Error("Cannot delete the last profile")
  delete d.profiles[name]
  if (d.active === name) {
    const rest = Object.keys(d.profiles).sort()
    d.active = rest[0]!
  }
  await writeSddModels(filepath, d)
}
```

```jsonc
// .opencode/sdd-models.jsonc — initial structure
{
  "active": "balanced",
  "profiles": {
    "balanced": {
      "sdd-apply":    "mistral/codestral-latest",
      "sdd-archive":  "opencode/mimo-v2-omni-free",
      "sdd-design":   "anthropic/claude-sonnet-4-6",
      "sdd-explore":  "mistral/mistral-large-2512",
      "sdd-init":     "opencode-go/kimi-k2.5",
      "sdd-propose":  "mistral/mistral-large-2512",
      "sdd-spec":     "opencode-go/glm-5",
      "sdd-tasks":    "google/gemini-3-flash-preview",
      "sdd-verify":   "opencode/nemotron-3-super-free"
    },
    "quality": {},
    "economy": {}
  }
}
```

---

## Builtin Restriction Removal — Exact Code Diff

### `sdd-models-default.ts` — remove the guard

```diff
-/** Names shipped with the default template — not removable via the TUI. */
-export const SDD_BUILTIN_PROFILE_NAMES = ["balanced", "quality", "economy"] as const
-
-export function isSddBuiltinProfile(name: string) {
-  return (SDD_BUILTIN_PROFILE_NAMES as readonly string[]).includes(name)
-}
-
 /** Default `.opencode/sdd-models.jsonc` when created from the TUI via `/sdd-models`. */
 export const SDD_MODELS_DEFAULT = `{ ... }`
```

### `sdd-models-file.ts` — remove guard at line 4 (import) and line 111

```diff
-import { SDD_MODELS_DEFAULT, isSddBuiltinProfile } from "./sdd-models-default"
+import { SDD_MODELS_DEFAULT } from "./sdd-models-default"
 ...
 export async function deleteSddProfile(filepath: string, name: string) {
-  if (isSddBuiltinProfile(name)) throw new Error(`Built-in profile "${name}" cannot be deleted`)
   const d = await readSddModels(filepath)
```

### `dialog-sdd-models.tsx` — remove guard at line 26 (import) and lines 89–91

```diff
-import { isSddBuiltinProfile } from "@tui/util/sdd-models-default"
 ...
         onTrigger: (option) => {
           if (option.value === NEW_SENTINEL) { ... return }
           const name = option.value
-          if (isSddBuiltinProfile(name)) {
-            toast.show({ message: `Built-in profile "${name}" cannot be deleted`, variant: "warning" })
-            return
-          }
           void (async () => {
```

---

## Testing Strategy

| Layer | What to Test | Approach |
|-------|-------------|----------|
| Unit | `deleteSddProfile` no longer throws for "balanced"/"quality"/"economy" | Jest/Vitest test calling the function with builtin names; expect no error thrown |
| Unit | `normalizeProfileName` accepts `[a-zA-Z0-9._-]`, rejects others | Existing tests remain valid |
| Integration | TUI delete keybind on a builtin profile name triggers confirm dialog, not warning toast | Playwright/teatest: simulate `ctrl+d` on "balanced" row, expect DialogConfirm, not toast |
| E2E | `/sdd-models` → profile deleted → persisted to `.opencode/sdd-models.jsonc` | Manual test with local binary build |

---

## Migration / Rollout

1. **Config artifacts** are safe to deploy immediately — the runtime ignores unknown files.
2. **Runtime patch** requires recompiling `lightcode-main` (`bun run build` from `packages/opencode`).
3. No data migration needed — existing `.opencode/sdd-models.jsonc` files remain valid.
4. Users on old binary still see the deletion warning but can edit the file directly.

---

## Open Questions

- [ ] Does the lightcode-main binary in use match the source in `lightcode-main`? Confirm before applying source patches.
- [ ] Should `quality` and `economy` profiles be pre-populated with cost/quality-optimized variants, or stay empty as per spec?
