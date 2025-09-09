# Foundation Donation Tracking System - Problem Formulation

> **⚠️ ARCHIVED DOCUMENT**  
> This document has been superseded by the [Foundation System Overview](foundation_system_overview.md) which reflects the current implemented architecture and design decisions.  
> This document is preserved for historical context of the original problem analysis.

## Current Situation

The foundation has 3 employees and tracks hundreds of donations per year using Podio. The current system uses separate apps for Needs, Transactions, People, Schools, and Groups, but is experiencing significant limitations.

### Current Entities
- **Needs**: What is required and for whom
- **Transactions**: Records of money sent  
- **People**: Individuals with immutable and mutable properties
- **Schools**: Institutions with immutable properties
- **Groups**: Sets of people (families, student cohorts)

### Current Workflow
1. Employee discovers a Need and creates it in Podio
2. Employee chooses who will pay (foundation or employee personally)
3. Employee categorizes Need (accepted, rejected, etc.)
4. Employee sends money to intermediary and creates Transaction record
5. Employee manually links Transaction to Need (bidirectional)
6. Need is marked as Paid

## Refined Understanding: Three-Tier Architecture

### NeedPackage (Group Level)
**What it represents:** A cohesive request made to the foundation, potentially involving multiple recipients.

**Key characteristics:**
- Represents **external requests** at the conceptual level
- Can have status: New, Accepted, Partially Accepted, Declined, Paid  
- Contains total **requested amount/scope** (preserved for audit trail)
- Distribution Type: Group-Only, Individual-Equal, Individual-Custom
- Provides clean UI view while enabling detailed tracking

### Need (Individual Level)
**What it represents:** A specific funding requirement for one recipient within a NeedPackage.

**Key characteristics:**
- Represents **individual recipient commitments** within the broader request
- Links to: NeedPackage + Person/School/Group
- Contains specific amount for this recipient
- Status derives from funding completion: Pending, Paid
- Enables precise per-recipient accountability
- Key insight: Same person can have multiple Needs across different NeedPackages

### Transaction  
**What it represents:** A financial transfer event - money actually moving from the foundation.

**Key characteristics:**
- Represents the **banking/logistics** level  
- Pure operational record of funds movement
- Success/failure based on banking/transfer mechanics
- Must equal the sum of related allocation amounts (critical constraint)

### Allocation
**What it represents:** The assignment of a specific transaction amount to fulfill a specific Need.

**Key characteristics:**
- Links Transactions to Needs with exact amounts
- One allocation per Need-Transaction pairing
- Enables partial funding: large Needs can have multiple Allocations
- Transaction Amount must equal Sum of related Allocation Amounts
- Provides perfect money traceability from transaction to final recipient

## Core Problems Identified

### 1. Many-to-Many Relationship Complexity
The system struggles with real-world scenarios where Needs and Transactions don't have 1:1 relationships:

**Multiple Needs, One Transaction (cost efficiency):**
- Need A: 100,000
- Need B: 200,000  
- Transaction limit: 150,000
- Most efficient: Two 150,000 transactions
- Transaction 1: 100% of Need A (100,000) + 25% of Need B (50,000)
- Transaction 2: 75% of Need B (150,000)

**One Need, Multiple Transactions (amount limits):**
- Need: 300,000
- Transaction limit: 150,000  
- Requires: Two 150,000 transactions

### 2. Many-to-Many Relationship Complexity
The system struggles with real-world scenarios where Needs and Transactions don't have 1:1 relationships:

**Multiple Needs, One Transaction (cost efficiency):**
- Need A: 100,000
- Need B: 200,000  
- Transaction limit: 150,000
- Most efficient: Two 150,000 transactions
- Transaction 1: 100% of Need A (100,000) + 25% of Need B (50,000)
- Transaction 2: 75% of Need B (150,000)

**One Need, Multiple Transactions (amount limits):**
- Need: 300,000
- Transaction limit: 150,000  
- Requires: Two 150,000 transactions

### 3. Three-Tier Granularity Requirements
Foundation needs different levels of tracking and accountability:

**NeedPackage Level (UI/Planning)**: Clean conceptual view
- Example: "30 students request transport - medium distance"  
- Shows in main UI as single line item
- Contains total requested amount and distribution approach

**Need Level (Individual Accountability)**: Per-recipient tracking  
- Example: "John's transport need within medium distance package" - 16,000 KES
- Enables precise individual spending accountability
- Same person can have multiple Needs across different NeedPackages
- Critical insight: John can have separate transport Need (16,000) AND calculator Need (1,000)

**Allocation Level (Transaction Traceability)**: Money flow precision
- Example: "10,000 KES from Transaction 2 applied to John's transport Need"
- Enables partial funding: John's 16,000 Need might require multiple Allocations
- Perfect audit trail from transaction to final recipient

**Key Insight**: This three-tier approach solves the UI pollution vs. precision tracking dilemma while maintaining perfect financial accountability.

### 4. UI Pollution vs. Precision Tracking
**The Dilemma**: Example F (20 students, varying travel amounts)

**Current Options:**
- Create 20 separate Needs â†’ Accurate tracking but UI becomes unusable
- Create 1 aggregate Need â†’ Clean UI but lose individual accountability

**Requirement**: Need both clean UI AND precise individual tracking

### 5. Amount Consistency Requirements
**Critical Constraint**: Transaction amounts must equal the sum of related allocation amounts for financial accountability.

**Three-Layer Amount Semantics:**
- **NeedPackage Amount**: Original total request (preserved for audit)
- **Need Amount**: Individual recipient requirement within package  
- **Allocation Amount**: Specific transaction assignment to Need
- **Transaction Amount**: Actual money sent

**Example Flow:**
- NeedPackage: 500,000 KES requested (30 students transport)
- Need: John's portion = 16,000 KES
- Allocations: Transaction 2 â†’ 10,000 KES + Transaction 4 â†’ 6,000 KES
- **Requirement**: Sum of John's Allocations (16,000) = John's Need Amount (16,000) âœ“

### 6. Data Duplication and Inconsistency
- Recipients duplicated in both Needs and Transactions
- Bidirectional linking requires manual double-entry (Transactionâ†’Need AND Needâ†’Transaction)
- No single source of truth for recipient information
- Linking often done incompletely, affecting automated analysis

### 7. Recurring Need Management
- No systematic way to handle recurring needs (monthly wages, semester fees)
- Manual recreation leads to inconsistency and administrative overhead
- Need templates vs. need instances conceptual gap

### 8. Status Management Complexity

**Need Status Ambiguity:**
- Multiple transactions for one need with mixed success rates
- Example: Need requires 100, Transaction A (50) succeeds, Transaction B (50) fails â†’ What's the Need status?

**Transaction Intent vs. Reality:**
- Need A: 100, Need B: 200
- Transaction 1 (150): Intended for A(100) + B(50), but fails
- Transaction 2 (150): Intended for B(150), succeeds  
- Question: Is Need A paid (money available) or unpaid (no specific allocation)?

**Group Status Complexity:**
- Student A, B, C selected for summer camp (3,000 each, total 9,000)
- Money sent (9,000), but Student A falls sick and can't attend
- What's the Need status? What happens to Student A's 3,000?

### 9. Reporting and Analysis Limitations
Current system cannot easily answer:
- "How much has Person X received total across all needs?"
- "What's our monthly spending on wages vs. one-time needs?"
- "Which needs are partially fulfilled and by how much?"
- "Where did Transaction Y's money actually go?"

### 10. Podio Platform Constraints
- Cannot use internal item IDs for CSV imports (must use text matching)
- Bulk operations limited  
- No sophisticated import-to-create-related-records functionality
- Need titles must be unique to avoid CSV import errors

## Real-World Examples That Must Be Handled

### Example A: Simple Individual Need
- School St. Andrews needs 100,000 for a new chalkboard

### Example B: Recurring Individual Need  
- Person Mrs. Stevens needs her February wage of 20,000 (monthly recurring)
- What happens when wage increases next year?

### Example C: Recurring Individual Need with Variation
- Person Mary needs to pay her 2nd semester fees of 100,000 
- Related to 1st semester but amount varies year to year

### Example D: Recurring Group-Collective Need
- Group "Jacobs Family" needs 20,000 every month for food

### Example E: One-time Group-Collective Need
- Group "Smith Family" needs 100,000 to repair their house
- Not divisible among family members

### Example F: Group-Distributed Variable Need (Complex)
- Group "Medium Distance Students" need 100,000 for holiday travel
- 20 students, amounts vary per student (details in spreadsheet)
- Student 1: 4,500, Student 2: 5,200, Student 3: 4,800, etc.

### Example G: Group-Distributed Equal Need
- Group "Students St. Stevens Primary" need 100,000 for summer camp
- Equal distribution among all participants

### Example J: Complex Three-Tier Scenario (Canonical)
**Multi-package scenario with overlapping recipients:**

**Day 1:** 30 students request transport (medium distance) - 500,000 KES total, Individual-Custom distribution
- Results in: 1 NeedPackage + 30 individual Needs with varying amounts

**Same day:** 10 students from St Michaels 6th grade request calculators - 10,000 KES, Individual-Equal distribution  
- Results in: 1 NeedPackage + 10 individual Needs of 1,000 KES each
- **Key complexity**: 2 students (John, Mary) appear in BOTH packages = 4 separate Needs total

**Day 2:** St Jane's school requests sanitary pads - 20,000 KES, Group-Only distribution
- Results in: 1 NeedPackage + 1 individual Need

**Transaction execution:** 4 planned transactions (150K each + 80K), but 2 fail
- Successful: Transaction 2 (150K) + Transaction 4 (80K) = 230K total sent
- Creates multiple Allocations linking successful transactions to specific Needs
- **Final result**: Some Needs fully funded, others pending, perfect audit trail maintained

**Demonstrates**: Same recipients across multiple NeedPackages, partial funding capability, transaction failure handling, perfect money traceability

## New Requirements Discovered

### 1. Intent Preservation
The system must preserve the original intent even when circumstances change:
- If money is sent but recipient becomes unavailable, maintain record of both the transfer and the changed circumstance
- Don't lose audit trail when plans change after money is in motion

### 2. Flexible Status Inheritance
- Transaction Status (banking/transfer success)
- Need Status (business requirement fulfillment)  
- Individual Allocation Status (recipient-specific situations)
- These statuses must be related but independently manageable

### 3. Backward Compatibility
- Existing data must be preservable
- Some short-term workflow pain is acceptable for long-term improvement

### 4. Naming Convention Solution
- Need titles must be unique for CSV import functionality
- Proposed format: "Description Month Year" (e.g., "Mrs Stevens Monthly Wage Feb 2025")

### 5. Money Trail Accountability  
System must always be able to account for:
- Where every sent dollar ended up
- Why any money is "in limbo" or unallocated
- The complete chain from original need through final recipient

## Success Criteria

A successful solution must:
1. Handle all example scenarios (A-J) elegantly
2. Maintain clean UI while preserving detailed tracking
3. Support complex transaction-to-need relationships
4. Provide accurate, comprehensive reporting
5. Work within Podio's platform constraints
6. Preserve complete audit trails
7. Minimize repetitive manual work
8. Scale to hundreds of needs/transactions per year