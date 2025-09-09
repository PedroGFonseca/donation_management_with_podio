# [Concept Name]

## What It Is
[One sentence definition]

## Purpose
[Why this concept exists in the system]

## Why it is needed 

## Key Relationships
- **References**: [Entities this app has relationship fields for]
- **Referenced by**: [Apps that have relationship fields pointing to this entity]
- **Implements**: [Many-to-many relationships this entity enables] (junction entities only)

## Fields
<!-- Use - **Name** (Type): description-->
<!-- For instance - **Amount** (Money): Amount needed for this recipient -->

<!-- IMPORTANT: All concepts with automation MUST include System Messages field -->
<!-- IMPORTANT: All status fields MUST specify automation rules and override scenarios -->
<!-- IMPORTANT: Use [Concept](concept.md) format for cross-references within concepts/ directory -->

### User Input Fields

### Mixed Fields
- **Status** (Single Select): [Status options]
  - **Automation**: [When automation applies and what it does]
  - **Manual override**: [When manual override is allowed and what it can change to]
  - **Override scenarios**: [Specific cases where manual override is appropriate]
  - **Constraint**: [Any business rules that prevent invalid status combinations]

### Strictly Automated Fields

### Buttons/Bulk Actions

### System Fields
- **System Messages** (Text Area): Automation log for [specific automation events for this concept]
  - Example: "[Date]: [Event description] - [Specific action taken]"
  - Example: "[Date]: [Event description] - [Specific action taken]"
  - Example: "[Date]: [Event description] - [Specific action taken]"


## Rules
[Critical business rules and constraints]

## Examples
[1-2 concrete examples]