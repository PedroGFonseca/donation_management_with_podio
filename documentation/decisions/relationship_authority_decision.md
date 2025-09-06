# Single-Side Relationship Authority - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

Podio apps frequently contain bidirectional relationships where the same logical connection can be expressed from either side. For example, [Person](../concepts/person.md)↔Family relationships can be managed through either the "Family" field on [Person](../concepts/person.md) records or the "Members" field on Family records. This creates data integrity risks and user confusion when both sides can be independently modified.

## The Bidirectional Relationship Problem

### Technical Issue
Podio relationship fields are unidirectional - they do not automatically synchronize with their counterpart fields. When two apps have fields pointing to each other, they operate as completely independent relationships despite representing the same logical connection.

### Data Integrity Scenarios

**Example: Person↔Family Relationship**

**Scenario A: Person-side entry**
- User creates Mary Thomas
- Sets Mary's "Family" field to "Thomas Family"
- Result: Mary shows connection to Thomas Family
- Problem: Thomas Family's "Members" field does NOT automatically include Mary

**Scenario B: Family-side entry**  
- User goes to Thomas Family record
- Adds Mary to the "Members" field
- Result: Thomas Family shows Mary as member
- Problem: Mary's "Family" field does NOT automatically point to Thomas Family

### Resulting Data Inconsistencies
- **One-way relationships**: Mary thinks she belongs to Thomas Family, but Thomas Family doesn't list Mary
- **Relationship asymmetry**: Reports and queries return different results depending on which field is checked
- **User confusion**: Team members unsure which field contains authoritative data
- **Maintenance overhead**: Manual effort required to keep both sides synchronized
- **Query complexity**: Applications must check both sides to find complete relationship data

## Decision: Single-Side Authority Model

**For every bidirectional relationship type in the foundation system, exactly one side will be designated as the authoritative source. The other side will be either read-only, informational, or unused.**

## Implementation Principles

### Authority Designation
Each app-to-app relationship will explicitly designate which side has authority:
- Authority side: Users can create, modify, and delete relationships
- Non-authority side: Field becomes informational or unused

### Consistency Enforcement
- Non-authoritative fields will be removed from apps entirely (since Podio lacks read-only field functionality)
- Replaced with calculation fields that display relationship information in read-only format (see [Calculation Fields for Bidirectional Relationship Display](calculation_fields_decision.md) decision document)
- User interfaces and training must clearly direct users to the authoritative side for relationship management
- Relationship queries should use only the authoritative field

### Common Relationship Patterns

**Container Relationships**: When one entity serves as a container for others
- Pattern: Authority typically resides with the container
- Examples: [School](../concepts/school.md) contains Students/Staff, Family contains Members
- Rationale: Container management is more natural from the container perspective

**Attribution Relationships**: When one entity belongs to or is assigned to another  
- Pattern: Authority typically resides with the item being categorized
- Examples: [Need](../concepts/need.md) belongs to Recipient, [Transaction](../concepts/transaction.md) belongs to [FundingBatch](../concepts/fundingbatch.md)
- Rationale: Assignment is more natural when creating the item being assigned

## Examples Across Foundation Apps

### Person↔Family Relationships
- **Authority**: [Person](../concepts/person.md).Family field ([Person](../concepts/person.md)-side authority)
- **Non-authority**: Family.Members field (calculation field showing related people)
- **Rationale**: Family assignment happens during person creation/updates; individual assignment more natural

### Person↔School Relationships  
- **Authority**: Person.School field (Person-side authority)
- **Non-authority**: School would not have a "Students" field, or it would be informational
- **Rationale**: School assignment happens during person creation/updates; individual assignment more natural

### Need↔Recipient Relationships
- **Authority**: Need.Recipient field (Need-side authority)  
- **Non-authority**: Person/School/Group would not have "Needs" fields, or they would be informational
- **Rationale**: Recipient assignment happens during need creation; individual assignment more natural

## Benefits of Single-Side Authority

### Data Integrity
- Eliminates relationship asymmetry
- Single source of truth for each relationship
- Consistent query results regardless of approach
- Reduced data validation complexity

### User Experience  
- Clear workflows for relationship management
- Reduced confusion about where to make changes
- Consistent training and documentation
- Predictable system behavior

### System Maintenance
- Simplified relationship queries
- Reduced need for synchronization logic
- Clearer audit trails through SystemDiffs
- Lower risk of relationship corruption

## Implementation Requirements

### Per-App Documentation
Each app must clearly specify:
- Which relationships it has authority over
- Which relationships are managed by other apps
- Correct workflows for relationship management
- Query patterns for relationship data

### User Interface Design
- Guide users to authoritative fields
- Make non-authoritative fields clearly informational
- Provide easy navigation to authoritative management interfaces
- Include helpful guidance about where to manage relationships

### Training and Procedures
- Document relationship authority patterns
- Train team on correct relationship management workflows
- Establish procedures for correcting relationship asymmetries
- Monitor SystemDiffs for incorrect relationship usage

## Migration and Validation

### Existing Data Assessment
- Audit current bidirectional relationships for asymmetries
- Identify which side contains more complete/accurate data
- Document current team usage patterns

### Data Consolidation
- Migrate relationship data to authoritative side
- Validate relationship completeness and accuracy
- Clear or mark non-authoritative fields as informational

### Ongoing Monitoring
- Use SystemDiffs to track relationship changes
- Monitor for accidental usage of non-authoritative fields
- Regular validation of relationship data integrity

## Success Criteria

This approach succeeds if:
- Relationship asymmetries eliminated across all apps
- User confusion about relationship management reduced
- Data integrity improved through single source of truth
- System queries return consistent results
- Maintenance overhead for relationship management decreased
- Clear documentation and workflows established

---

*This decision establishes a foundation-wide pattern for managing bidirectional relationships, with specific authority designations to be determined per app based on operational workflows and user patterns.*