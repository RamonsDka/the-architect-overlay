# Profile Storage Specification

## Purpose

Specification for the `.opencode/sdd-models.jsonc` file format and persistence mechanics for SDD model profiles.

## Requirements

### Requirement: File Location

The system SHALL store SDD model profiles in `.opencode/sdd-models.jsonc` relative to project root.

#### Scenario: File Search Path

- GIVEN the system needs to load SDD model profiles
- WHEN searching for configuration
- THEN the system searches upward from current working directory
- AND finds the first `.opencode/sdd-models.jsonc` file encountered

#### Scenario: File Creation on First Use

- GIVEN no `.opencode/sdd-models.jsonc` file exists
- WHEN user first invokes `/sdd-models add <profile-name>` or `/sdd-models list`
- THEN the system creates a new file with default structure
- AND includes three empty default profiles: "balanced", "quality", "economy"
- AND sets "balanced" as the active profile

### Requirement: File Format

The system MUST use JSONC (JSON with Comments) format for the profile storage file.

#### Scenario: Valid JSONC Parsing

- GIVEN a `.opencode/sdd-models.jsonc` file exists
- WHEN the system reads the file
- THEN the system SHALL parse it as JSONC (allowing trailing commas and comments)
- AND extract `active` (string) and `profiles` (object) fields

#### Scenario: Invalid File Handling

- GIVEN a `.opencode/sdd-models.jsonc` file has invalid JSON/JSONC syntax
- WHEN the system attempts to parse it
- THEN the system SHALL throw an error with message "Invalid JSON in [filepath]"
- AND prevent further profile operations until file is fixed

#### Scenario: Missing Fields Handling

- GIVEN a `.opencode/sdd-models.jsonc` file lacks required fields
- WHEN the system reads it
- THEN the system SHALL default missing `active` field to "balanced"
- AND default missing `profiles` field to an empty object

### Requirement: Profile Structure

Each profile SHALL be a map of agent names to model references.

#### Scenario: Profile Data Model

- GIVEN a profile named "custom-profile"
- WHEN accessing profile data
- THEN each entry is formatted as `"agent-name": "provider/model"`
- AND empty profiles (`{}`) ARE valid and inherit from agent defaults

#### Scenario: Model Reference Parsing

- GIVEN a model reference string "provider/model-id"
- WHEN the system parses it
- THEN the system splits on the first "/" character
- AND extracts providerID and modelID separately
- AND rejects invalid formats (empty provider/model, no slash)

### Requirement: Profile Name Validation

The system SHALL validate profile names according to specific character restrictions.

#### Scenario: Valid Characters

- GIVEN user is creating a profile via `/sdd-models add <name>`
- WHEN the name is validated
- THEN the system allows ONLY letters (a-z, A-Z), digits (0-9), underscores (_), dots (.), and hyphens (-)
- AND maximum length is 64 characters

#### Scenario: Invalid Name Rejection

- GIVEN user provides a profile name
- WHEN the name contains invalid characters or exceeds length
- THEN the system SHALL reject the name
- AND display error message "Invalid name (letters, digits, ._- only, max 64 chars)"

### Requirement: Default Profiles Creation

The system SHALL create three default profiles when initializing a new file.

**NOTE: These profiles are deletable just like any other profile. No special protection.**

#### Scenario: First-Time File Initialization

- GIVEN no `.opencode/sdd-models.jsonc` exists
- WHEN the system initializes it
- THEN the file contains profiles "balanced", "quality", and "economy"
- AND all profiles have empty model assignments (`{}`)
- AND "balanced" is set as active profile

```jsonc
{
  "active": "balanced",
  "profiles": {
    "balanced": {},
    "quality": {},
    "economy": {}
  }
}
```

## Persistence Requirements

### Requirement: Atomic File Writes

The system MUST write profile changes atomically to prevent corruption.

#### Scenario: Write Operation

- GIVEN user makes a profile change via any `/sdd-models` subcommand
- WHEN the system persists to disk
- THEN the system SHALL write the complete JSON structure
- AND append newline character at end of file
- AND use pretty-print formatting (2-space indentation)

#### Scenario: Concurrent Access Protection

- GIVEN multiple processes may access the file
- WHEN writing profile changes
- THEN the system SHALL use atomic write (write to temp, then rename)
- AND prevent data loss from concurrent modifications