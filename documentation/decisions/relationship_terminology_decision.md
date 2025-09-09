# Relationship Terminology for App Documentation - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

The foundation system documentation needs consistent terminology to describe relationships between entities across different Podio apps. The current "parent-child" terminology incorrectly implies hierarchical ownership where none exists, and fails to clarify where relationship fields physically reside and can be modified.

## Relationship Types in the Foundation System

### Actual Relationship Patterns
The foundation system contains three primary relationship patterns:

**Association/Reference Relationships**
- Need → Person (recipient assignment)
- Person → School (affiliation)
- Allocation → FundingBatch (funding source)

**Organizational Containment**
- NeedPackage contains multiple Needs
- FundingBatch organizes multiple Allocations

**Junction/Associative Relationships**
- Allocation implements FundingBatch ←→ Need many-to-many relationship

### The Authority vs. Logic Distinction

**Logical Relationships**: Conceptual data model connections
- Need conceptually belongs to a Person
- Person conceptually has multiple Needs

**Physical Implementation**: Where relationship fields actually exist in Podio
- Need app contains "Recipient" field pointing to Person
- Person app contains NO "Needs" field (per Single-Side Authority decision)

## Decision: References/Referenced By Terminology

### Core Principle
**"References" indicates physical field authority, not just logical relationships.**

### Terminology Standards

**References**: Relationships where this app contains the physical field and has modification authority
- Maps directly to relationship fields that exist in this app
- Indicates where users must go to create/modify the relationship
- Aligns with Single-Side Relationship Authority decision

**Referenced by**: Relationships where other apps contain fields pointing to this entity
- Indicates this entity serves as a target for other entities' relationship fields
- Users cannot modify these relationships from this app
- Provides context about how this entity is used by the system

**Implements**: Applies only to junction entities that enable many-to-many relationships
- Describes associative entities that resolve complex relationships
- Includes the specific many-to-many relationship being implemented

## Implementation Standards

### Documentation Template
```markdown
## Key Relationships
- **References**: [Entities this app has relationship fields for]
- **Referenced by**: [Apps that have relationship fields pointing to this entity]
- **Implements**: [Many-to-many relationships this entity enables] (junction entities only)
```

### Foundation System Examples

**Need:**
```markdown
## Key Relationships
- **References**: Person/School/Group (recipient field), NeedPackage (needpackage field)
- **Referenced by**: Allocation records (need field in Allocation app)
```

**Person:**
```markdown
## Key Relationships
- **References**: School (school field), Group (family field)
- **Referenced by**: Need records (recipient field in Need app)
```

**Allocation:**
```markdown
## Key Relationships
- **References**: FundingBatch (fundingbatch field), Need (need field)
- **Referenced by**: None
- **Implements**: FundingBatch ←→ Need many-to-many relationship
```

**NeedPackage:**
```markdown
## Key Relationships
- **References**: None (top-level request entity)
- **Referenced by**: Need records (needpackage field in Need app)
```

**FundingBatch:**
```markdown
## Key Relationships
- **References**: None (top-level funding entity)
- **Referenced by**: Allocation records (fundingbatch field in Allocation app)
```

## Benefits of This Approach

### Operational Clarity
- Users know exactly which app contains modifiable relationship fields
- Clear mapping between documentation and actual Podio app structure
- Eliminates confusion about where to make relationship changes

### Consistency with Existing Decisions
- Aligns with Single-Side Relationship Authority decision
- Reinforces established patterns for relationship management
- Supports unidirectional linking strategy

### Standard Database Terminology
- Uses established Entity-Relationship modeling concepts
- Familiar to developers and system analysts
- Avoids inventing custom terminology for well-known patterns

### Documentation Accuracy
- Physical implementation matches logical description
- No ambiguity about field locations
- Clear operational guidance for users

## Migration from Current Documentation

### Replace "Parent/Children" Language
- Remove hierarchical implications where none exist
- Replace with appropriate relationship type (References, Referenced by, Implements)
- Maintain existing relationship directions based on field authority

### Audit Existing Relationship Descriptions
- Verify that "References" only includes fields this app actually contains
- Ensure "Referenced by" accurately reflects other apps' relationship fields
- Add "Implements" notation for junction entities like Allocation

### Update User Training Materials
- Emphasize field authority patterns
- Clarify where users should go to modify specific relationships
- Reinforce Single-Side Authority workflows

## Success Criteria

This terminology succeeds if:
- Documentation accurately reflects Podio app field structure
- Users can quickly identify where to modify relationships
- Relationship descriptions are consistent across all app documentation
- New team members can understand relationship management without confusion
- System complexity is reduced through clear operational guidance

---

*This decision establishes foundation-wide terminology for relationship documentation that maps directly to Podio's physical implementation while using standard database concepts.*