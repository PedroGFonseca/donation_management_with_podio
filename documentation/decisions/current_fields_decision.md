# Current-State Fields Policy - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

Foundation Podio apps have accumulated time-bound historical fields (e.g., "Received support in 2024", "Grade/year in Jan 2025", "Has received support in 2025") that create UI clutter, maintenance overhead, and data integrity issues. The question is whether to continue storing historical data in individual app fields or adopt a different approach.

## Decision: Focus Apps on Current State Only

**All Podio apps will store only current-state information. Historical tracking will be handled through the SystemDiffs and MonthlyBackups infrastructure.**

## Rationale

This decision is based on comprehensive analysis documented in the "SystemDiffs and MonthlyBackups - Foundation System Design Decision" which established that reliable historical tracking requires dedicated infrastructure independent of Podio's limitations.

### Primary Technical Justification

**Podio's Historical Tracking Limitations:**
- 30-revision API limit causes permanent data loss for frequently-updated records
- No guaranteed long-term data retention post-account termination
- Webhook unreliability makes real-time historical capture unsuitable for audit compliance
- UI subject to same revision limitations as API

**SystemDiffs Infrastructure Provides Superior Solution:**
- 100% change detection through dual-capture approach (Podio revisions + state comparison)
- 7-year ANBI-compliant retention independent of Podio account status
- Attribution tracking when available, graceful degradation when not
- Complete audit trail across all apps in unified structure

## Fields to Remove from Apps

### Person App Time-Bound Fields
- ~~"Received support in 2024"~~ → Query SystemDiffs/Allocation records
- ~~"Has received support in 2025"~~ → Query SystemDiffs/Allocation records
- ~~"Grade/year in Jan 2025"~~ → Use current "Grade/Year" field only

### General Pattern for All Apps
Remove any field with year/date specificity that represents historical snapshots rather than current state.

## Benefits of Current-State Focus

### UI and Usability Improvements
- **Cleaner interfaces**: Apps show only relevant, current information
- **Reduced cognitive load**: Users focus on current state without historical clutter
- **Simplified data entry**: No need to update multiple year-specific fields
- **Better mobile experience**: Fewer fields improve mobile app usability

### Data Integrity Benefits
- **Single source of truth**: Current state stored once, not duplicated across time-bound fields
- **Automatic consistency**: No risk of contradictory values between current and historical fields
- **Reduced manual maintenance**: No annual field updates required
- **Clear data ownership**: Current state in apps, history in SystemDiffs

### Operational Efficiency
- **Simplified workflows**: Team focuses on current decisions, not historical bookkeeping
- **Faster queries**: Current state queries don't compete with historical data retrieval
- **Reduced training burden**: New team members learn simpler app structures
- **Lower error rates**: Fewer fields means fewer opportunities for data entry mistakes

### Audit and Compliance Advantages
- **Superior historical tracking**: SystemDiffs provides complete change history with attribution
- **Regulatory compliance**: MonthlyBackups ensure ANBI 7-year retention requirements
- **Investigation capability**: Can reconstruct any historical state through SystemDiffs
- **Cross-app correlation**: Unified historical tracking enables system-wide analysis

## Implementation for Specific Apps

### Person App
**Keep (Current State)**:
- Current grade/year, Current school, Current status (in touch/lost touch)
- Current priority, Demographics (name, DOB, gender, etc.)
- Current contact information, Current family relationships

**Remove (Historical)**:
- All year-specific support tracking fields
- Historical grade/year fields
- Time-bound status or priority fields

**Historical Queries Available Through SystemDiffs**:
- Support received by year → Query Allocation records + SystemDiffs
- Grade progression → SystemDiffs field changes over time
- Status change history → SystemDiffs status field changes

### School App
**Keep (Current State)**:
- Current school information, Current contact details
- Current status, Current vetting status

**Remove (Historical)**:
- Historical relationship status fields
- Year-specific operational notes

### Need/NeedPackage Apps
**Keep (Current State)**:
- Current status (New, Allocated, Paid, etc.)
- Current amounts and recipients
- Current priority and deadlines

**Remove (Historical)**:
- Historical status progression fields
- Year-specific categorizations

## Historical Data Access Strategy

### For Operational Queries
**Recent History (1-2 years)**: Direct SystemDiffs queries for day-to-day operational needs

**Example Queries**:
- "Show all grade changes for Person X in last year"
- "Who changed Need Y status last month?"
- "What support did Person Z receive this year?"

### For Audit and Compliance
**Long-term History (7+ years)**: Combination of MonthlyBackups (baselines) + SystemDiffs (changes)

**Example Reconstruction**:
- Start with monthly backup nearest to target date
- Apply SystemDiffs forward or backward to exact date
- Generate complete historical state for audit purposes

### For Investigation and Analysis
**Cross-app Analysis**: SystemDiffs enables correlation across all apps

**Example Analysis**:
- "Show all system changes on Day X" (investigate issues)
- "Track Person Y across all apps over time" (comprehensive history)
- "Correlate Need status changes with Allocation creation" (workflow analysis)

## Migration Strategy

### Phase 1: SystemDiffs Implementation
- Deploy SystemDiffs and MonthlyBackups infrastructure
- Begin capturing all changes across existing apps
- Validate historical tracking accuracy

### Phase 2: App Simplification
- Remove time-bound fields from apps systematically
- Migrate any critical historical queries to SystemDiffs-based reports
- Train team on new historical query methods

### Phase 3: Validation and Optimization
- Verify historical reconstruction accuracy
- Optimize SystemDiffs queries for common use cases
- Document new procedures for audit and compliance

## Success Criteria

This approach succeeds if:
- Apps become cleaner and more focused on current operations
- Historical tracking needs met through SystemDiffs queries
- Audit and compliance requirements satisfied through new infrastructure
- Team productivity improves due to simplified app interfaces
- Data integrity improves through single-source-of-truth approach
- ANBI compliance maintained through superior historical tracking

## Relationship to Other Decisions

This decision is enabled by and depends on the "SystemDiffs and MonthlyBackups" infrastructure decision, which provides:
- Complete change detection and attribution
- Long-term historical storage independent of Podio
- Reconstruction capability for any historical date
- Audit-compliant retention and reporting

---

*This policy transforms Podio apps from historical data repositories into clean, current-state operational tools while ensuring superior historical tracking through dedicated infrastructure.*