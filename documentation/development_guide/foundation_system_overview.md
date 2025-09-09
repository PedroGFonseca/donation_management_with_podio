# Foundation Donation Management System - Current Architecture Overview

*System Overview - September 2025*

## System Purpose

The Foundation Donation Management System is a comprehensive Podio-based solution for tracking donations, managing funding requests, and ensuring complete financial accountability for a small foundation with 3 employees handling hundreds of donations annually.

## Core Architecture

### Entity Hierarchy

The system implements a sophisticated three-tier architecture that balances operational efficiency with detailed financial accountability:

```
NeedPackage (Group Level)
    ↓ contains
Need (Individual Level) 
    ↓ funded by
Allocation (Junction Entity)
    ↑ sources from
FundingBatch (Funding Level)
```

### Key Entities

#### **NeedPackage** - Group Request Management
- **Purpose**: Organizes related funding requests into manageable units
- **Key Features**: 
  - Clean UI for bulk operations
  - Multiple distribution types (Group-Final, Individual-Equal, Individual-Custom)
  - Spreadsheet import capability for bulk need creation
  - Status aggregation from child Needs
- **Relationships**: Referenced by Need records (needpackage field)

#### **Need** - Individual Accountability
- **Purpose**: Atomic unit of funding requirement for specific recipients
- **Key Features**:
  - Precise per-recipient tracking
  - Status automation based on allocation completion
  - Support for multiple recipients (Person/School/Group)
  - Cross-package recipient tracking
- **Relationships**: References Person/School/Group (recipient field), NeedPackage (needpackage field); Referenced by Allocation records

#### **Allocation** - Funding Assignment
- **Purpose**: Junction entity solving many-to-many funding relationships
- **Key Features**:
  - Explicit money assignment from FundingBatch to Need
  - Auto-allocation for bulk operations
  - Partial funding support
  - Perfect audit trail maintenance
- **Relationships**: References FundingBatch (fundingbatch field), Need (need field); Implements FundingBatch ←→ Need many-to-many relationship

#### **FundingBatch** - Funding Source Management
- **Purpose**: Organizes funding sources and enables bulk allocation
- **Key Features**:
  - Auto-allocation when amounts match exactly
  - Manual mode for complex scenarios
  - Credit tracking and validation
  - Exception handling and logging
- **Relationships**: Referenced by Allocation records (fundingbatch field)

#### **Person** - Individual Recipients
- **Purpose**: Current-state demographic and role information
- **Key Features**:
  - Current information only (historical tracking via SystemDiffs)
  - Status automation based on recent activity
  - School and family group affiliations
- **Relationships**: References School (school field), Group (family field); Referenced by Need records

#### **School** - Educational Institutions
- **Purpose**: Institution-level recipient management
- **Key Features**: [To be defined based on requirements]

#### **Group** - Collective Recipients
- **Purpose**: Family units and organizational groups
- **Key Features**:
  - Final Receiver property for distribution type
  - Membership snapshot for individual distribution
  - Support for both collective and individual funding patterns
- **Relationships**: [To be defined based on actual implementation]

## Advanced System Features

### Auto-Allocation System

**Purpose**: Eliminates manual work for bulk funding operations while maintaining control and accuracy.

**How it works**:
1. User creates FundingBatch and links to NeedPackage(s)
2. System calculates: `sum(linked Need amounts) vs FundingBatch amount`
3. **If amounts match exactly**: Auto-creates Allocation records for all linked Needs
4. **If amounts don't match**: Switches to manual mode, requires user decision

**Exception Handling**:
- Switches to manual mode when Need amounts change after allocation
- Provides clear System Messages explaining why auto-allocation stopped
- Allows user to recalculate allocations after fixing issues

### Group Distribution Logic

**Two Distribution Patterns**:

1. **Final Receiver Groups** (Final Receiver = True):
   - Groups receive funding as collective entities
   - Single Need created for entire group
   - Examples: Families, organizations

2. **Individual Distribution Groups** (Final Receiver = False):
   - Groups used to create individual Needs for each member
   - Membership snapshot at creation time
   - Members tracked individually for accountability
   - Examples: Student classes, member lists

### Field Control System

**Four Field Types**:

1. **User Input Fields**: Full human control (titles, descriptions, priorities)
2. **Mixed Fields**: Automation with manual override capability (status fields)
3. **Strictly Automated Fields**: System-only control (financial calculations)
4. **System Fields**: Platform-managed (timestamps, IDs)

### Relationship Management

**Single-Side Authority Model**:
- Each relationship has exactly one authoritative side
- Non-authoritative fields removed to prevent data inconsistency
- Clear operational guidance on where to modify relationships

**Relationship Terminology**:
- **References**: Entities this app has relationship fields for
- **Referenced by**: Apps that have relationship fields pointing to this entity
- **Implements**: Many-to-many relationships this entity enables (junction entities only)

### Audit and Compliance

**SystemDiffs Infrastructure**:
- Captures all field changes with attribution
- 7-year retention for ANBI compliance
- Independent of Podio account status
- Complete change detection and audit trail

**MonthlyBackups**:
- Full record snapshots for disaster recovery
- Cross-app change correlation
- Historical reconstruction capability

## Operational Workflows

### Bulk Need Creation
1. User creates spreadsheet with recipient details and amounts
2. System imports spreadsheet and creates NeedPackage + individual Needs
3. User reviews and finalizes package
4. System generates individual Need records for perfect accountability

### Funding Allocation
1. User creates FundingBatch with available funds
2. User links FundingBatch to NeedPackage(s)
3. **Auto-allocation**: System creates Allocations automatically if amounts match
4. **Manual allocation**: User creates Allocations manually for complex scenarios
5. System tracks allocation status and updates Need statuses accordingly

### Status Management
- **Need Status**: Auto-calculated based on allocation completion
- **NeedPackage Status**: Aggregated from child Need statuses
- **Allocation Status**: Tracks funding success/failure
- **Exception Handling**: System Messages field logs all automation decisions

## Key Design Decisions

### 1. Three-Tier Architecture
**Decision**: Separate NeedPackage (UI), Need (accountability), and Allocation (traceability) levels
**Benefit**: Clean UI management with detailed individual tracking

### 2. Auto-Allocation with Manual Override
**Decision**: Automated bulk operations with human override capability
**Benefit**: Operational efficiency while maintaining control for exceptions

### 3. Single-Side Relationship Authority
**Decision**: Each relationship has exactly one authoritative side
**Benefit**: Prevents data inconsistency and user confusion

### 4. Group Distribution Patterns
**Decision**: Final Receiver property determines distribution behavior
**Benefit**: Clear, predictable group behavior for different use cases

### 5. Field Control Levels
**Decision**: Four distinct field types with different control mechanisms
**Benefit**: Financial integrity protection with operational flexibility

### 6. SystemDiffs Audit Infrastructure
**Decision**: Comprehensive change tracking independent of Podio limitations
**Benefit**: ANBI compliance and complete audit trail

## Real-World Scenarios Handled

### Scenario 1: Student Transport (Bulk Individual)
- 50 students need transport with varying costs
- NeedPackage created with Individual-Custom distribution
- 50 individual Needs generated from spreadsheet
- FundingBatch auto-allocates to all 50 Needs

### Scenario 2: Family House Repair (Collective)
- Smith Family needs house repair funding
- Group with Final Receiver = True
- Single Need created for entire family
- Funding allocated to family unit

### Scenario 3: Cross-Batch Funding
- Large Need requires multiple funding sources
- Multiple Allocations from different FundingBatches
- System tracks total allocation vs Need amount
- Status updates when fully funded

### Scenario 4: Partial Funding
- Need requires 100,000 KES
- First Allocation: 60,000 KES (successful)
- Second Allocation: 40,000 KES (failed)
- System shows "Partially Paid" status

## Technical Implementation

### Podio Platform Constraints
- Unidirectional relationships (accepted and designed around)
- Limited bulk operations (mitigated with automation)
- 30-revision API limit (addressed with SystemDiffs)
- No true button functionality (simulated with dropdown fields)

### Automation Architecture
- Webhook-based status inheritance
- API-driven bulk operations
- Exception logging and user notification
- Conservative dual-capture audit strategy

### Data Integrity Measures
- Strictly automated financial calculations
- Single-side relationship authority
- Complete audit trail maintenance
- Exception handling and logging

## Success Metrics

The system successfully addresses all original requirements:
- ✅ Handles all example scenarios (A-J) elegantly
- ✅ Maintains clean UI while preserving detailed tracking
- ✅ Supports complex transaction-to-need relationships
- ✅ Provides accurate, comprehensive reporting
- ✅ Works within Podio's platform constraints
- ✅ Preserves complete audit trails
- ✅ Minimizes repetitive manual work
- ✅ Scales to hundreds of needs/transactions per year

---

*This overview reflects the current state of the Foundation Donation Management System as of September 2025, incorporating all major design decisions and architectural choices.*
