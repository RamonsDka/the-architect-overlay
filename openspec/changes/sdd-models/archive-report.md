# Archive Report: /sdd-models Profile System Port

**Status**: ✅ ARCHIVED & COMPLETED
**Date**: 2026-04-03
**Mode**: Hybrid (Engram + OpenSpec)

## Executive Summary
The `/sdd-models` profile system from `lightcode-main` has been successfully ported to the `opencode` project. The primary goal of transitioning to a CLI-first integration while maintaining TUI capabilities, alongside the critical requirement of **allowing the deletion of built-in profiles**, has been fully achieved.

## Phase Consolidation

1. **sdd-explore**: Investigated the TUI-based SolidJS implementation in `lightcode-main`. Identified the rigid `isSddBuiltinProfile` checks protecting `balanced`, `quality`, and `economy`.
2. **sdd-propose**: Proposed a two-track strategy: add local config artifacts to `opencode` while patching the `lightcode-main` runtime code to strip out the deletion restrictions.
3. **sdd-spec**: Defined formal specifications for the new behavior. Ensured the rule "Built-in Profiles Are Deletable" was codified, with the only protection being that the *last* profile cannot be deleted.
4. **sdd-design**: Designed the exact technical patches required (removing lines in `sdd-models-default.ts`, `sdd-models-file.ts`, and `dialog-sdd-models.tsx`).
5. **sdd-tasks**: Broke down the design into a granular 9-step checklist.
6. **sdd-apply**: Executed the code changes:
   - Created `.opencode/sdd-models.jsonc` seeding current agent models to the `balanced` profile.
   - Created `commands/sdd-models.md` to trigger the TUI.
   - Removed the `isSddBuiltinProfile` guard logic and the UI toast warning from `lightcode-main`.
7. **sdd-verify**: Audited the filesystem and code. Cleaned up a residual dangling import. Verified that the restriction is fully gone.

## State of the System
- **Profiles**: The system starts with `balanced`, `quality`, and `economy`.
- **Flexibility**: The user has absolute freedom to delete **any** of these default profiles via the UI (`Ctrl+D`).
- **Safety**: The system prevents the deletion of the final profile to avoid empty states.

## Conclusion
The `sdd-models` port is officially closed and archived. The implementation meets all user requirements without any unresolved technical debt.