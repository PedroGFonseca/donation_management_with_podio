# Historical Tracking Strategy - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

Foundation system entities contain both current state information (Jon is currently in 3rd grade at School B) and historical tracking needs (Jon was in 2nd grade at School A in 2024, moved to 3rd grade same school in 2025, then transferred to School B while repeating 3rd grade in 2026). The question is whether to store historical data explicitly in separate fields/apps or implement a comprehensive historical tracking system.

## Technical Discovery

Podio API provides field history tracking through item revisions but has critical limitations:
- **Get Item Revisions**: `/item/{item_id}/revision` - Returns revisions with timestamps and users
- **Get Revision Differences**: `/item/{item_id}/revision/{from}/{to}` - Shows field changes between revisions
- **30-revision API limit**: Only returns most recent 30 revisions, older changes permanently lost
- **No guaranteed retention**: Data may be lost if Podio account terminated

## Decision: Comprehensive SystemDiffs + MonthlyBackups Strategy

**We will implement a dual-capture historical tracking system that combines Podio API revisions with daily state comparison, stored in dedicated SystemDiffs and MonthlyBackups apps.**

**This approach uses Podio API as one component of a more comprehensive solution documented in [SystemDiffs and MonthlyBackups - Foundation System Design Decision](diff_storage_decision.md).**

## Dual-Capture Implementation

### SystemDiffs Component
The comprehensive solution implements daily capture using both:
1. **Podio API revisions**: For detailed attribution and granular changes
2. **State comparison**: For 100% change detection as safety net

### Current-State App Design
**All apps focus on current state only**:
- ~~"Received support in 2024"~~ → Query SystemDiffs/[Allocation](../concepts/allocation.md) records
- ~~"Grade/year in Jan 2025"~~ → Use current "Grade/Year" field only
- ~~Historical status fields~~ → Query SystemDiffs for progression

**Focus on operational efficiency**:
- Current grade/year, school, status, priority
- Demographics (name, DOB, gender, etc.)
- Current contact information and relationships

### Historical Access Pattern
**Operational Queries** (recent history):
```python
# Grade progression via SystemDiffs
changes = query_systemdiffs(app="Person", item_id="123", field="Grade")
# Result: 2024: Form 2 → 2025: Form 3 → 2026: Form 3

# Support history via [Allocation](../concepts/allocation.md) records + SystemDiffs
support_2024 = query_allocations(recipient="Person123", year=2024)
```

**Audit Reconstruction** (long-term history):
```python
# Combine MonthlyBackups baseline + SystemDiffs changes
state_2025_jan = nearest_monthly_backup("2025-01") + apply_systemdiffs_since()
```

### Benefits of Integrated Approach

**Complete Coverage**:
- Podio API provides attribution and detailed change context
- State comparison ensures no changes ever lost (even if >30 revisions)
- MonthlyBackups provide baseline snapshots for reconstruction
- 7-year ANBI compliance guaranteed

**Operational Reliability**:
- Apps remain clean and current-state focused
- Historical queries available through SystemDiffs infrastructure
- No dependency on Podio revision limits or retention policies
- Cross-app change correlation and audit trails

**Financial Integrity**:
- Support history derived from authoritative [Allocation](../concepts/allocation.md) records
- Complete audit trail for all financial calculations
- Attribution tracking for both manual and automated changes

## Implementation Integration

**Daily Process**:
1. Read Podio API revisions (detailed attribution)
2. Perform state comparison (complete coverage)
3. Store both in SystemDiffs with full context
4. Parse System Messages for automation attribution

**Query Capabilities**:
- Recent changes: Direct SystemDiffs queries
- Historical reconstruction: MonthlyBackups + SystemDiffs
- Attribution analysis: Podio revision data in SystemDiffs
- Cross-app correlation: Unified SystemDiffs across all apps

## Success Criteria

This integrated approach succeeds if:
- Apps become cleaner and more current-state focused
- 100% change detection achieved across all apps
- Complete attribution preserved where available
- ANBI 7-year retention satisfied with full audit capability
- Historical reconstruction possible for any date
- Performance acceptable for operational needs

---

*This decision leverages Podio's built-in capabilities to simplify our data model while maintaining complete historical tracking and audit compliance.*