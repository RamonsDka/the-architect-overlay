# Profile Management Specification

## Purpose

Specification for CRUD operations on SDD model profiles with explicit requirement that ALL profiles (including built-in defaults) are deletable. No restrictions on deleting "balanced", "quality", or "economy" profiles.

## Requirements

### Requirement: Create Profile

The system SHALL allow users to create new profiles with optional cloning from existing profiles.

#### Scenario: Create Empty Profile

- GIVEN user invokes `/sdd-models add <profile-name>`
- WHEN the profile name is valid and unique
- THEN the system creates an empty profile (no model assignments)
- AND sets the new profile as active
- AND persists to `.opencode/sdd-models.jsonc`

#### Scenario: Create Profile Cloning Current

- GIVEN user invokes `/sdd-models add <profile-name> --clone <source-profile>`
- WHEN the source profile exists
- THEN the system creates a new profile copying ALL settings from source
- AND sets the new profile as active

#### Scenario: Duplicate Name Rejection

- GIVEN user provides a profile name that already exists
- WHEN the system attempts to create it
- THEN the system SHALL reject with error "Profile '[name]' already exists"
- AND preserve existing profile unchanged

### Requirement: List Profiles

The system SHALL display all available profiles with their model assignments.

#### Scenario: Display All Profiles

- GIVEN user invokes `/sdd-models list`
- WHEN the command executes
- THEN the system displays all profile names
- AND marks the active profile with `[active]` or similar indicator
- AND for each profile, shows agent → model assignments

### Requirement: Switch Profile

The system SHALL allow selecting any existing profile as the active profile.

#### Scenario: Profile Switch

- GIVEN user invokes `/sdd-models switch <profile-name>`
- WHEN the profile exists
- THEN the system updates the `active` field in `.opencode/sdd-models.jsonc`
- AND displays confirmation "Switched to profile '[name]'"
- AND all subsequent agent model lookups use the new active profile

### Requirement: Set Agent Model

The system SHALL allow updating model assignments for specific agents within the active profile.

#### Scenario: Set Agent Model to Profile

- GIVEN user invokes `/sdd-models set <agent-name> <model-reference>`
- WHEN the agent name is a valid SDD agent
- THEN the system adds/updates the model assignment for that agent in the active profile
- AND persists changes to `.opencode/sdd-models.jsonc`
- AND displays confirmation "[agent] → [provider/model]"

#### Scenario: Clear Agent Model Assignment

- GIVEN an agent has a model assignment in the active profile
- WHEN user invokes `/sdd-models set <agent-name> --clear`
- THEN the system removes that agent's entry from the profile
- AND the agent inherits from `agent.model` config or session model

### Requirement: Delete Profile

**CRITICAL: This requirement explicitly allows deletion of ALL profiles, including built-in defaults.**

The system SHALL allow users to delete ANY profile, including "balanced", "quality", and "economy" profiles. The `isSddBuiltinProfile()` restriction is REMOVED.

#### Scenario: Delete Any Profile

- GIVEN user invokes `/sdd-models delete <profile-name>`
- WHEN the profile exists and is not the last profile
- THEN the system prompts for confirmation "Delete '[name]'?"
- AND upon confirmation, deletes the profile from `.opencode/sdd-models.jsonc`
- AND displays confirmation "Deleted [name]"

#### Scenario: Delete Default Profiles Allowed

- GIVENprofiles "balanced", "quality", or "economy" exist
- WHEN user invokes `/sdd-models delete balanced` (or quality/economy)
- THEN the system SHALL allow the deletion WITHOUT restrictions
- AND NO warning about "built-in profile" is displayed
- AND NO error "Built-in profile '[name]' cannot be deleted" is thrown
- AND NO check for `isSddBuiltinProfile()` is performed

#### Scenario: Last Profile Protection

- GIVEN only one profile remains in the system
- WHEN user attempts to delete it
- THEN the system SHALL reject with error "Cannot delete the last profile"
- AND preserve the remaining profile

#### Scenario: Active Profile Deletion

- GIVEN user deletes the currently active profile
- WHEN the deletion completes
- THEN the system selects the first remaining profile (alphabetically sorted) as active
- AND updates the `active` field accordingly

### Requirement: Model Resolution Hierarchy

The system SHALL resolve agent models using a specific precedence order.

#### Scenario: Model Resolution Order

- GIVEN an SDD agent needs a model assignment
- WHEN the system resolves the model
- THEN the resolution follows this order:
  1. Profile override: `profiles[active][agentName]`
  2. Agent config: `agent[agentName].model`
  3. Session fallback: primary session model
  4. Empty: no model assigned

#### Scenario: Empty Profile Fallback

- GIVEN an active profile has no model assignments (`{}`)
- WHEN SDD agents run
- THEN each agent uses its configured `model` from `opencode.json`
- OR falls back to the primary session model if not configured

## Error Handling

### Requirement: Graceful Error Handling

The system SHALL provide clear error messages for all failure scenarios.

#### Scenario: File Read Failure

- GIVEN `.opencode/sdd-models.jsonc` is corrupted or unreadable
- WHEN the system attempts to load profiles
- THEN the system displays error message with details
- AND continues operation using default profile structure in memory
- AND does not crash the application

#### Scenario: File Write Failure

- GIVEN a file system error prevents writing
- WHEN the system attempts to save changes
- THEN the system displays error message with details
- AND preserves previous state in memory
- AND does not corrupt existing file

#### Scenario: Invalid Model Reference

- GIVEN user provides an invalid model reference format
- WHEN user invokes `/sdd-models set <agent> <invalid-model>`
- THEN the system displays error "Invalid model reference. Format: provider/model"
- AND does not modify the profile