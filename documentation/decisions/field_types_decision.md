# Field Types and Control Levels - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

Foundation system entities have fields controlled by different actors (users, automation, system). We need clear categorization to define:
- What users can edit vs what they cannot
- Which fields require strict automation enforcement
- How to handle conflicts between automation and manual overrides

## Field Type Categories

### 1. User Input Fields
**Definition**: Fields that users manually enter, select, or modify
**Control**: Humans have full control
**System Role**: None (except validation)

**Examples**:
- **Title** (NeedPackage): User creates descriptive name
- **Description** (Need): User provides context details  
- **Priority** (Need): User sets urgency level
- **Recipients** (NeedPackage): User selects who will benefit
- **Notes** (all entities): User adds operational observations

**Why users control these**: These fields contain business context, strategic decisions, and operational knowledge that only humans possess.

### 2. Mixed Fields  
**Definition**: Fields calculated by automation but can be manually overridden by users
**Control**: Automation sets initial value, humans can override when needed
**System Role**: Calculate default value, log when humans override

**Examples**:
- **Status** (Need): Auto-calculated as "Paid" when allocations sum >= amount, but user can override to "Cancelled" if recipient unavailable
- **Status** (NeedPackage): Auto-calculated based on child Need statuses, but user can override for operational exceptions
- **Type Summary** (NeedPackage): Auto-generated breakdown, but user might add manual notes for context

**Why mixed control**: Automation handles the common case efficiently, but real-world exceptions require human judgment (recipient unavailable, changed circumstances, operational complexities).

### 3. Strictly Automated Fields
**Definition**: Fields that must maintain system-calculated values for financial integrity
**Control**: System only - humans forbidden from manual changes
**System Role**: Calculate and enforce correct values, revert manual changes, log override attempts

**Examples**:
- **Total Amount** (NeedPackage): Must equal sum(child Need amounts)
- **Allocated Amount** (Need): Must equal sum(successful Allocation amounts)
- **Available Credit** (FundingBatch): Must equal funded_amount - allocated_amount
- **Need Count** (NeedPackage): Must equal actual count of child Needs
- **Unique Recipients** (NeedPackage): Must equal distinct recipient count

**Why strictly automated**: These fields ensure financial consistency and audit integrity. Manual changes would:
- Break accounting accuracy
- Violate ANBI compliance requirements
- Create discrepancies that could hide errors or fraud
- Undermine trust in financial reporting

### 4. System Fields
**Definition**: Standard platform tracking fields
**Control**: System only (platform-managed)
**System Role**: Automatic timestamping and tracking

**Examples**:
- **Created Date**: When record was created
- **Last Updated**: When record was last modified
- **Record ID**: Unique system identifier

**Why system-controlled**: These provide audit trail metadata managed by the platform itself.

## Implementation Rules

### Webhook Enforcement for Strictly Automated Fields
When a user manually changes a strictly automated field:
1. **Immediate correction**: Webhook recalculates and restores correct value
2. **Logging**: Record the manual change attempt with user, timestamp, and incorrect value
3. **Alert**: Notify administrators of the override attempt
4. **No exceptions**: Financial integrity fields cannot be manually overridden under any circumstances

### Mixed Field Override Protocol
When a user overrides a mixed field:
1. **Allow the change**: Accept user's operational judgment
2. **Log the override**: Record that automation was overridden with reason
3. **Preserve automation**: Continue calculating what the automated value would be
4. **Clear indication**: UI shows when field is manually overridden vs automated

### User Input Field Validation
User input fields should have:
- **Format validation**: Ensure data types and formats are correct
- **Business rule validation**: Check constraints (positive amounts, valid dates)
- **No automation**: System never changes user-entered values without explicit user action

## Benefits of This Approach

### Financial Integrity Protection
Strictly automated fields prevent accidental or intentional corruption of financial calculations that are critical for:
- ANBI compliance and audit requirements
- Accurate donor reporting
- Cross-foundation financial accuracy
- Detection of allocation errors

### Operational Flexibility
Mixed fields allow humans to handle real-world exceptions while maintaining automated efficiency for common cases.

### Clear User Expectations
Users understand exactly what they can control vs what the system manages, reducing confusion and accidental errors.

### Audit Trail Completeness
Logging all override attempts provides complete visibility into when and why automation was bypassed.

## Examples in Practice

### Scenario 1: Student Becomes Unavailable
- **Need Status**: Auto-calculated as "Paid" based on successful allocation
- **Real situation**: Student fell sick, couldn't attend funded event
- **User action**: Override status to "Cancelled" 
- **System response**: Accept "Cancelled" as new status, Podio logs the field change automatically

### Scenario 2: User Tries to "Fix" Amounts
- **Total Amount**: Shows 150,000 KES (sum of child needs)
- **User concern**: Thinks amount should be 160,000 KES
- **User action**: Attempts to manually change Total Amount
- **System response**: Immediately revert to 150,000 KES, log the attempt, alert administrators
- **Correct action**: User should check/correct individual Need amounts, Total Amount will recalculate automatically

### Scenario 3: Operational Context Addition
- **Description field**: User adds details about recipient situation
- **System response**: Accept the input, no automation involvement
- **Benefit**: Preserves important operational context for future reference

---

*This categorization ensures financial integrity while maintaining operational flexibility and clear user expectations.*