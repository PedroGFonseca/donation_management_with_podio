# SystemDiffs and MonthlyBackups - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

Foundation system requires reliable historical tracking across all Podio apps to meet ANBI 7-year audit requirements. The challenge is ensuring complete change detection and attribution while working within Podio's limitations (30-revision API limit, no guaranteed long-term retention, unreliable webhooks).

## Core Requirements

**Audit Compliance:**
- 7-year retention requirement for ANBI status
- Complete change detection (no lost changes)
- Attribution tracking when possible
- Independence from Podio account status

**Operational Needs:**
- Historical reconstruction capability
- Investigation of specific changes
- Cross-app change correlation
- Performance suitable for daily operations

## Technical Constraints Discovered

**Podio API Limitations:**
- Maximum 30 revisions returned per item
- 5,000 revision hard limit per item  
- No guaranteed data retention post-termination
- UI subject to same revision limits as API

**Webhook Reliability Issues:**
- 15-second timeout with automatic suspension
- Permanent disabling after repeated failures
- Low priority queue with throttling delays
- Not suitable for critical audit compliance

## Decision: Dual Storage Approach

**We will implement SystemDiffs + MonthlyBackups apps using a conservative dual-capture strategy.**

## SystemDiffs App Design

### Purpose
Capture individual field changes with attribution when available, ensuring no changes are ever lost regardless of Podio limitations.

### Structure
```
- App (Text): Source app name ("Person", "School", "Need", etc.)
- Item ID (Text): Specific record that changed
- Field Name (Text): Which field changed
- Change Date (Date): When change occurred  
- Old Value (Text): Previous value
- New Value (Text): New value
- Changed By (Person): Who made change (null when unavailable)
- Source (Select): "Podio Revision", "Daily State Comparison"
- Attribution Available (Checkbox): Whether "who changed" is known
```

### Daily Capture Process
1. **Podio revision reading**: Store all revisions from last 24 hours (Source="Podio Revision")
2. **State comparison**: Compare current snapshots vs yesterday's snapshots (Source="Daily State Comparison")
3. **Accept duplication**: Both sources stored, deduplication handled at analysis time

## MonthlyBackups App Design

### Purpose
Provide complete baseline snapshots for reconstruction and audit compliance, independent of Podio revision limitations.

### Structure
```
- App (Text): Source app name
- Month (Date): Backup month/year
- CSV Data (File): Complete app export as CSV
- Record Count (Number): Number of records in backup
- Backup Date (Date): When backup was generated
```

### Monthly Process
Automated export of complete current state for all apps as CSV files, stored in MonthlyBackups app.

## Rejected Approaches and Rationale

### Approach 1: Podio-Only Historical Tracking
**Concept**: Rely solely on Podio's revision system for historical data
**Rejection Reasons**: 
- 30-revision limit causes permanent data loss
- No guaranteed long-term retention
- ANBI compliance risk unacceptable

### Approach 2: Webhook-Primary Capture
**Concept**: Use webhooks as primary change capture mechanism
**Rejection Reasons**:
- Webhooks unreliable (15-second timeout, automatic suspension)
- Events lost during suspension periods
- Permanent disabling after repeated failures
- Cannot serve as foundation for audit compliance

### Approach 3: Smart Detection of API Incompleteness
**Concept**: Detect when Podio API returns incomplete data (30+ revisions) and only then add state comparison diffs
**Rejection Reasons**:
- False positives: Items with exactly 30 revisions flagged incorrectly
- False negatives: Items with 31+ revisions not detected
- Complex logic built on undocumented API behavior
- Maintenance burden when Podio changes behavior

### Approach 4: Intra-day History Embedding
**Concept**: Store granular revision sequences within daily state comparison records
**Rejection Reasons**:
- Multi-user attribution ambiguity (User A: 2→3, User B: 3→4 = who changed 2→4?)
- Round-trip activity loss (User A: 2→3, User B: 3→2 = no detected change)
- Complex data structures defeat simplicity of state comparison
- Query complexity compounds over time

### Approach 5: App-Specific Historical Storage
**Concept**: Create PersonHistory, SchoolHistory, NeedHistory apps
**Rejection Reasons**:
- Unsustainable proliferation of apps
- No unified audit trail across system
- Complex queries spanning multiple history apps
- Maintenance overhead scales with number of entity types

### Approach 6: Separate Snapshot Schedules by App
**Concept**: Different backup frequencies based on app change patterns
**Rejection Reasons**:
- Implementation complexity without significant benefit
- Storage savings minimal (total <400MB over 7 years)
- Operational confusion about which apps use which approach
- Uniform approach provides better audit consistency

## Benefits of Chosen Approach

**Complete Audit Coverage:**
- SystemDiffs ensures no changes ever lost (state comparison safety net)
- MonthlyBackups provide complete baseline snapshots
- Attribution captured when available (Podio revisions)
- 7-year retention guaranteed independent of Podio

**Operational Reliability:**
- Daily batch processing (no webhook dependency)
- Conservative dual capture eliminates edge case failures
- Simple analysis-time filtering rather than complex capture logic
- Uniform approach across all apps

**Reconstruction Capability:**
- Historical state = Nearest monthly backup ± SystemDiffs
- Maximum 30 days of diffs to apply for any reconstruction
- Human-readable CSV backups for manual audit review
- Complete audit trail for regulatory compliance

## Implementation Requirements

**Daily Automation:**
- Snapshot all app current states
- Read Podio revisions for all apps (last 24 hours)
- Calculate state diffs vs previous day
- Store both revision and state comparison data in SystemDiffs

**Monthly Automation:**
- Export all apps as CSV
- Store CSV files in MonthlyBackups app
- Verify backup completeness and integrity

**Analysis Capabilities:**
- Detailed activity queries: Filter Source="Podio Revision"
- Complete coverage queries: Deduplicate across both sources
- Attribution gap analysis: Identify "Daily State Comparison" without corresponding "Podio Revision"
- Historical reconstruction: Combine monthly baselines with SystemDiffs

## Storage Projections

**SystemDiffs Volume (7 years):**
- Estimated 500-800 actual changes per year across all apps
- 3,500-5,600 total diff records
- Well under Podio's 500,000 item limit per app

**MonthlyBackups Volume (7 years):**
- ~6 apps × 12 months × 7 years = 504 backup records
- Total CSV storage: ~300-400MB
- Negligible storage impact

## Success Criteria

This approach succeeds if:
- 100% change detection achieved across all apps
- Attribution captured for 99%+ of normal operations (under 30 revisions/day)
- ANBI 7-year retention requirement satisfied
- Historical reconstruction possible for any date
- System resilient to Podio account issues
- Implementation complexity remains manageable
- Audit queries performant and comprehensive

---

*This conservative dual-capture approach prioritizes audit compliance and operational reliability over optimization, ensuring no changes are ever lost while maintaining attribution when possible.*