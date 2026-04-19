# CLI Commands Specification

## Purpose

Specification for the `/sdd-models` CLI command with subcommands for SDD model profile management. This is a CLI-first implementation with optional TUI support.

## Requirements

### Requirement: Command Registration

The system SHALL provide a `/sdd-models` slash command with subcommands for profile management.

#### Scenario: List Profiles

- GIVEN user invokes `/sdd-models list`
- WHEN the command executes
- THEN the system displays all available profiles
- AND highlights the currently active profile
- AND shows model assignments for each agent in each profile

#### Scenario: Switch Active Profile

- GIVEN user invokes `/sdd-models switch <profile-name>`
- WHEN the profile exists
- THEN the system sets that profile as active
- AND persists the change to `.opencode/sdd-models.jsonc`
- AND displays confirmation "Switched to profile '[name]'"

#### Scenario: Add New Profile

- GIVEN user invokes `/sdd-models add <profile-name>`
- WHEN the name is valid and doesn't exist
- THEN the system creates an empty profile
- AND sets it as active
- AND displays confirmation "Created profile '[name]'"

#### Scenario: Add Profile Cloning

- GIVEN user invokes `/sdd-models add <profile-name> --clone <source-profile>`
- WHEN the source profile exists
- THEN the system creates a new profile copying all settings from source
- AND sets it as active
- AND displays confirmation "Created profile '[name]' from '[source]'"

#### Scenario: Delete Profile

- GIVEN user invokes `/sdd-models delete <profile-name>`
- WHEN the profile exists and is not the last profile
- THEN the system removes the profile
- AND persists changes to `.opencode/sdd-models.jsonc`
- AND displays confirmation "Deleted profile '[name]'"
- AND if deleted profile was active, switches to first remaining profile alphabetically

#### Scenario: Set Agent Model

- GIVEN user invokes `/sdd-models set <agent-name> <model-reference>`
- WHEN the agent name is a valid SDD agent
- THEN the system assigns the model to that agent in the active profile
- AND persists changes to `.opencode/sdd-models.jsonc`
- AND displays confirmation "[agent] → [provider/model]"

### Requirement: Subcommand Error Handling

The system SHALL provide clear error messages for invalid subcommand usage.

#### Scenario: Unknown Subcommand

- GIVEN user invokes `/sdd-models <unknown-subcommand>`
- WHEN the subcommand is not recognized
- THEN the system displays error "Unknown subcommand '[name]'. Available: list, switch, add, delete, set"
- AND shows usage help

#### Scenario: Missing Arguments

- GIVEN user invokes `/sdd-models switch` without profile name
- WHEN the command executes
- THEN the system displays error "Usage: /sdd-models switch <profile-name>"

#### Scenario: Profile Not Found

- GIVEN user invokes `/sdd-models switch <non-existent>`
- WHEN the profile doesn't exist
- THEN the system displays error "Profile '[name]' not found"
- AND lists available profiles

#### Scenario: Last Profile Deletion

- GIVEN only one profile remains
- WHEN user invokes `/sdd-models delete <that-profile>`
- THEN the system rejects with error "Cannot delete the last profile"
- AND preserves the remaining profile

### Requirement: TUI Mode Support

The system SHOULD support an interactive TUI mode when invoked without subcommands.

#### Scenario: Interactive Mode Invocation

- GIVEN user is in TUI mode
- WHEN user invokes `/sdd-models` without subcommand
- THEN the system opens an interactive profile management dialog
- AND displays current active profile and SDD agent model assignments
- AND allows CRUD operations through the UI

### Requirement: Built-in Profiles Are Deletable

**CRITICAL: This requirement explicitly removes all restrictions on deleting default profiles.**

The system SHALL allow deletion of ALL profiles, including "balanced", "quality", and "economy".

#### Scenario: Delete Default Profiles Without Warning

- GIVEN profiles "balanced", "quality", or "economy" exist
- WHEN user invokes `/sdd-models delete balanced` (or quality/economy)
- THEN the system deletes the profile WITHOUT "built-in profile" warning
- AND does NOT show warning about protected profiles
- AND does NOT check `isSddBuiltinProfile()` or equivalent

## Environment Variables

### Requirement: Environment Variable Override

The system SHALL support `OPENCODE_SDD_MODEL_PROFILE` environment variable override.

#### Scenario: Env Variable Takes Precedence

- GIVEN `OPENCODE_SDD_MODEL_PROFILE` is set to a profile name
- WHEN the system loads SDD model configuration
- THEN the system uses the specified profile regardless of `active` field in file
- AND does not modify the file's `active` field

## Help and Documentation

### Requirement: Help Command

The system SHALL provide help documentation for the `/sdd-models` command.

#### Scenario: Help Invocation

- GIVEN user invokes `/sdd-models --help` or `/sdd-models help`
- WHEN the command executes
- THEN the system displays:
  - Available subcommands: list, switch, add, delete, set
  - Usage syntax for each subcommand
  - Examples of common operations
  - Reference to `.opencode/sdd-models.jsonc` for manual editing