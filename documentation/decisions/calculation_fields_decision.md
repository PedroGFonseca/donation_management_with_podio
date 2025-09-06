# Calculation Fields for Bidirectional Relationship Display - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

When implementing single-side relationship authority to prevent data integrity issues, the non-authoritative side loses the ability to directly display relationship information. Users need to see relationships from both directions while maintaining data consistency and avoiding the bidirectional relationship problem.

## Solution: Calculation Fields for Read-Only Display

**Calculation fields can reference data from related apps and provide read-only displays of relationship information without creating data integrity risks.**

## Technical Capabilities

### Cross-App References
Podio calculation fields can reference fields in other apps that are connected via relationship fields. This enables displaying relationship data from the authoritative side on the non-authoritative side.

### Syntax and Examples
- **Basic reference**: `@All of FieldName` pulls data from related records
- **Text concatenation**: `@All of Name.join(", ")` creates comma-separated lists
- **Count relationships**: `@All of FieldName.length` shows number of related items

### Output Types
Calculation fields output one of three types determined at creation:
- **Number**: Counts, sums, calculations
- **Date**: Date calculations and formatting
- **Text**: Concatenated strings, formatted lists

## Implementation Pattern

### Authority Side
- Maintains the authoritative relationship field
- Users create, modify, and delete relationships here
- Field remains editable and functional

### Non-Authority Side  
- Remove original relationship field entirely
- Add calculation field that displays related records
- Calculation field is read-only by design
- Shows current state of relationships from authority side

### Example: [Person](../concepts/person.md)↔Family Relationships
```
Authority Side ([Person](../concepts/person.md) app):
- Family field: Editable relationship to Groups app

Non-Authority Side (Family app):
- Remove: Members relationship field  
- Add: Members calculation field
- Formula: @All of Name (where Name comes from [Person](../concepts/person.md)s with Family = this Family)
- Output: "Mary Thomas, John Thomas, Sarah Thomas"
```

## Benefits

### Data Integrity
- Single source of truth maintained (authority side only)
- Impossible to create relationship asymmetries
- No risk of inconsistent bidirectional data
- Automatic updates when authority side changes

### User Experience
- Relationships visible from both sides
- Clear indication of read-only status
- Better than Related Items clutter
- Users can spot names then navigate via Related Items for details

### Operational Safety
- No user training needed on "which side to edit"
- System prevents relationship editing from wrong side
- Calculation updates automatically with authority changes
- Reduced complexity in relationship management

## Limitations

### Display Functionality
- **Text-only output**: Cannot create clickable links to individual records
- **No direct navigation**: Users must use Related Items section to access individual records
- **Limited formatting**: Text concatenation with basic JavaScript formatting only

### Performance Considerations
- **Calculation queue delays**: Updates may not be immediate
- **Large dataset impact**: Performance may degrade with many relationships
- **Recalculation triggers**: Only updates when referenced fields change

### User Workflow Impact
- **Two-step navigation**: Spot name in calculation field, then find in Related Items
- **Less intuitive**: Not as seamless as native relationship field experience
- **Read-only limitation**: Cannot perform relationship actions from display side

## Use Cases and Applications

### Strong Fit
- **Container relationships**: Families containing members, Schools containing students
- **Attribution displays**: Showing "belongs to" information in read-only format
- **Relationship counts**: Displaying number of related items
- **Summary information**: Concatenated lists for overview purposes

### Weak Fit
- **Interactive relationship management**: Where users need to frequently add/remove relationships
- **Navigation-heavy workflows**: Where clicking through to individual records is primary use case
- **Complex relationship operations**: Bulk operations or sophisticated relationship filtering

## Implementation Requirements

### Field Setup
- Remove non-authoritative relationship fields completely
- Create calculation fields with appropriate formulas
- Set calculation output type correctly (cannot be changed later)
- Test calculation performance with expected data volumes

### User Training
- Document which side has authority for each relationship type
- Train users on two-step navigation (calculation field → Related Items)
- Establish procedures for relationship management workflows
- Set expectations about read-only nature of calculation displays

### Monitoring and Maintenance
- Monitor calculation field performance and queue delays
- Track user feedback on workflow changes
- Validate calculation formulas update correctly
- Document calculation field formulas for maintenance

## Success Criteria

This approach succeeds if:
- Relationship data integrity maintained through single authority
- Users can view relationships from both sides effectively
- Calculation fields provide sufficient relationship visibility
- Workflow disruption minimized compared to Related Items only
- System performance remains acceptable with calculation fields
- User confusion about relationship editing eliminated

## Decision Context

This solution addresses the core tension between data integrity and user experience in bidirectional relationships. While calculation fields provide less functionality than native relationship fields, they offer significant advantages over complete removal of non-authoritative fields and reliance solely on Related Items sections.

The trade-off prioritizes data integrity and system reliability while maintaining reasonable user experience for relationship visibility.

---

*This approach leverages Podio's calculation field capabilities to solve the bidirectional relationship problem while providing read-only relationship displays that support operational workflows.*