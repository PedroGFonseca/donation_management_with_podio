# Schema Evolution and Historical Tracking - Foundation System Design Decision

*Decision Document - September 2025*
**Status: UNRESOLVED - Requires further analysis and implementation design**

## Problem Statement

The SystemDiffs and MonthlyBackups historical tracking approach assumes stable Podio app schemas. However, operational reality requires periodic schema changes (adding fields, removing fields, modifying choice values). These changes create significant challenges for historical data integrity and reconstruction capability.

## Schema Change Scenarios Identified

### 1. Adding New Fields
**Problem**: 
- Yesterday's state snapshot lacks new field, today's snapshot includes it
- State comparison detects "change" from null → current value for every record
- Generates false positive SystemDiffs for every existing record

**Impact**: Massive noise in change detection, obscuring real data changes

### 2. Deleting Existing Fields  
**Problem**:
- Yesterday's snapshot contains field, today's snapshot doesn't
- State comparison detects "change" from previous value → null for every record
- Historical reconstruction impossible (field no longer exists in current schema)

**Impact**: Data appears lost, reconstruction capability compromised

### 3. Adding Multiple Choice Values
**Problem**: Minimal impact on existing data
**Assessment**: Current approach likely handles this adequately

### 4. Removing Multiple Choice Values
**Problem**:
- Records with deleted values may show null or error states
- State comparison detects value → null changes for affected records
- May not distinguish between deliberate data change and schema change

**Impact**: Schema changes appear as data corruption

### 5. Field Type Changes
**Problem**:
- Date field becomes text field, comparison logic breaks
- Value format changes appear as data changes ("2025-01-01" vs "January 1, 2025")
- Historical SystemDiffs become uninterpretable with current schema

**Impact**: Historical data becomes unusable for reconstruction

### 6. Field Renaming
**Problem**:
- Appears as deletion of old field + creation of new field
- All historical data appears lost for renamed field
- Relationship between old and new field names not captured

**Impact**: Historical continuity broken

## Current Approach Limitations

### No Schema Change Detection
- Daily process assumes stable app structure
- Cannot distinguish schema changes from data changes
- No mechanism to flag when comparison logic may be invalid

### Reconstruction Impossibility
- Cannot rebuild historical state when current schema differs from historical schema
- SystemDiffs reference fields that may no longer exist
- No schema versioning in MonthlyBackups

### Data Integrity Risks
- Schema changes create false positives in change detection
- Real data changes may be obscured by schema change noise
- Audit trail becomes unreliable during schema evolution periods

## Potential Approaches Under Consideration

### Approach A: Schema Versioning
**Concept**: Track app schema changes separately from data changes
**Requirements**:
- Detect schema changes before state comparison
- Version MonthlyBackups with schema metadata
- Store schema evolution history
- Enhance reconstruction logic to handle schema differences

**Challenges**:
- Complex detection logic for all schema change types
- Backward compatibility for historical reconstruction
- Podio API may not provide sufficient schema change detection

### Approach B: Schema Change Isolation
**Concept**: Pause historical tracking during schema change periods
**Requirements**:
- Manual flag when schema changes are planned
- Skip state comparison on flagged days
- Document schema changes in separate tracking system

**Challenges**:
- Manual intervention required
- Risk of missing data changes during schema change periods
- Doesn't solve reconstruction problems for historical data

### Approach C: Enhanced Change Detection
**Concept**: Improve daily process to distinguish schema vs data changes
**Requirements**:
- Pre-comparison schema validation
- Field-by-field change classification
- Separate storage for schema changes vs data changes

**Challenges**:
- Complex logic to detect all schema change scenarios
- May still miss edge cases
- Computational overhead for daily processing

### Approach D: Accept Limitations
**Concept**: Document schema change impacts and handle manually
**Requirements**:
- Clear procedures for schema changes
- Manual data validation during schema change periods
- Documentation of when historical reconstruction may be unreliable

**Challenges**:
- Manual overhead
- Risk of data integrity issues
- Limited historical reconstruction capability

## Critical Questions Requiring Resolution

### Technical Feasibility
- Can Podio API reliably detect schema changes before state comparison?
- What metadata is available about app structure changes?
- How can schema versions be efficiently stored and compared?

### Operational Impact
- How frequently do schema changes occur in foundation operations?
- What level of manual intervention is acceptable?
- Which schema change types are most critical to handle automatically?

### Audit Compliance
- Do ANBI requirements specify handling of schema evolution in historical records?
- What level of historical reconstruction capability is required for compliance?
- How should schema changes be documented for audit purposes?

### Implementation Complexity
- What is the development cost of each approach?
- How does schema handling affect the simplicity benefits of the current approach?
- What are the ongoing maintenance requirements for each solution?

## Impact on Existing Decisions

### SystemDiffs Design May Require Enhancement
Current structure may need additional fields:
- Schema Version (references when record was created)
- Change Type (Data Change vs Schema Change)
- Schema Change Details (what changed in app structure)

### MonthlyBackups May Require Metadata
- App schema snapshot alongside data snapshot
- Schema version tracking
- Change documentation

### Reconstruction Logic Complexity
- Historical state reconstruction becomes significantly more complex
- May require schema mapping tables for field evolution
- Backward compatibility requirements for old SystemDiffs

## Next Steps Required

1. **Schema Change Frequency Analysis**: Assess how often foundation apps undergo schema changes
2. **Podio API Schema Detection Research**: Investigate what schema change detection is possible
3. **Audit Requirement Clarification**: Determine compliance requirements for schema evolution handling
4. **Implementation Cost Analysis**: Evaluate development effort for each potential approach
5. **Pilot Testing**: Test chosen approach with controlled schema changes
6. **Decision Documentation**: Update this document with chosen approach and implementation plan

## Dependencies

This decision affects and depends on:
- SystemDiffs and MonthlyBackups infrastructure implementation
- Current-State Fields Policy implementation
- Future app evolution planning
- Audit and compliance procedures

---

**UNRESOLVED STATUS**: This document identifies critical gaps in the current historical tracking approach. Schema evolution handling must be resolved before full implementation of the SystemDiffs strategy, as schema changes could compromise the integrity of the entire historical tracking system.

*The foundation's historical tracking system cannot be considered complete until schema evolution scenarios are properly addressed.*