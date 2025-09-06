# Podio API Field History - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

Foundation system entities contain both current state information (Jon is currently in 3rd grade at School B) and historical tracking needs (Jon was in 2nd grade at School A in 2024, moved to 3rd grade same school in 2025, then transferred to School B while repeating 3rd grade in 2026). The question is whether to store historical data explicitly in separate fields/apps or leverage Podio's built-in revision tracking capabilities.

## Technical Discovery

Podio API provides comprehensive field history tracking through item revisions:
- **Get Item Revisions**: `/item/{item_id}/revision` - Returns all revisions with timestamps and users
- **Get Revision Differences**: `/item/{item_id}/revision/{from}/{to}` - Shows exactly which fields changed between specific revisions
- **Field-level tracking**: Can see old value → new value for any field change
- **Complete audit trail**: Who changed what when, with full context

## Decision: Use Podio API Field History

**We will rely on Podio's built-in revision system for historical tracking rather than creating explicit historical storage fields or apps.**

## Implications for System Design

### Person App Simplification
**Remove all time-bound fields**:
- ~~"Received support in 2024"~~ → Query Allocation records
- ~~"Has received support in 2025"~~ → Query Allocation records  
- ~~"Grade/year in Jan 2025"~~ → Use current "Grade/Year" field only

**Focus on current state**:
- Current grade/year
- Current school
- Current status (in touch/lost touch)
- Current priority
- Demographics (name, DOB, gender, etc.)

### Historical Queries Available Through API
**Grade progression tracking**:
```
2024: revision shows "Grade: Form 2, School: School A"
2025: revision shows "Grade: Form 3, School: School A" 
2026: revision shows "Grade: Form 3, School: School B"
```

**Status change tracking**:
```
2024: revision shows "Status: In touch, Priority: High"
2025: revision shows "Status: Lost touch, Priority: Low"
2026: revision shows "Status: In touch, Priority: Medium"
```

**Support history tracking**:
- Primary source: Allocation records (financial accuracy)
- Secondary verification: Person field changes if needed

### Benefits of This Approach

**Data Model Cleanliness**:
- Person app contains only current, relevant information
- No duplicate storage of historical data
- No artificial time-bound field proliferation
- No need for PersonYear or similar tracking apps

**Audit Compliance**:
- Complete field change history maintained automatically
- User attribution for every change
- Timestamp accuracy for all modifications
- No manual historical record maintenance required

**Operational Efficiency**:
- Single source of truth for current person data
- Historical queries available when needed via API
- No need to manually update year-specific fields
- Reduced data entry burden

**Financial Accuracy**:
- Support history derived from authoritative Allocation records
- Person app doesn't become source of truth for financial data
- Cross-validation possible between person changes and allocation timing

### Implementation Requirements

**API Integration**:
- Build Python functions to query revision history when needed
- Create utilities to show field progression over time
- Implement caching for frequently-accessed historical data

**Reporting Capabilities**:
- "Show Jon's school progression" → API revision query
- "Who changed status from In Touch to Lost Touch?" → Revision difference API
- "Support received by year" → Allocation query with person filter

**UI Considerations**:
- Current person data shows in main view
- Historical progression available via "Show History" feature
- Integration between person changes and allocation timeline

### Edge Cases and Limitations

**API Rate Limits**:
- Revision queries count against Podio API limits
- May need caching for frequently-accessed historical data
- Bulk historical analysis may require careful rate limit management

**Data Retention**:
- Podio retains revision history - verify retention policies align with ANBI requirements
- No control over Podio's revision retention period
- May need periodic export for long-term archival

**Query Complexity**:
- Simple current state queries remain fast
- Historical progression queries require API calls
- Complex historical analysis may be slower than dedicated historical tables

## Alternative Approaches Rejected

**PersonYear App**: Would duplicate data available through revisions while adding complexity

**Historical Fields**: Would clutter Person app with time-bound data that becomes obsolete

**Separate Historical Apps**: Would create data synchronization challenges and potential inconsistencies

## Validation Criteria

This approach succeeds if:
- Person app becomes cleaner and more focused
- Historical tracking needs are met through API queries
- API performance is acceptable for operational needs
- Audit and compliance requirements are satisfied through revision history
- Development complexity is reduced compared to custom historical storage

---

*This decision leverages Podio's built-in capabilities to simplify our data model while maintaining complete historical tracking and audit compliance.*