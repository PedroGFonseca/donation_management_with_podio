# NeedPackage

## What It Is
A group-level funding request that contains multiple individual [Need](need.md) records, providing clean UI management while maintaining detailed individual accountability.

## Purpose
To organize related funding requests into manageable units that can mix multiple recipients and multiple need types, while automatically generating precise individual [Need](need.md) records for perfect accountability.

## Why it is needed 
NeedPackage enables clean UI organization without sacrificing individual tracking. It allows natural bundling of requests (like "School F new semester needs") while ensuring every recipient and amount is precisely tracked through individual Need records.

## Key Relationships
- Parent: None (top-level request entity)
- Children: Individual [Need](need.md) records (one per recipient per need type)
- Links to: [Person](person.md)/[School](school.md)/[Group](group.md) records (recipients)

## Fields

### User Input Fields
- **Title** (Text): Descriptive name for this group request
- **Description** (Text Area): Overall description of the request and context
- **Currency** (Single Select): Currency for all amounts in this package (defaults to foundation's default currency)
- **Request Date** (Date): When this request was submitted
- **Deadline** (Date): When funding is needed by
- **Context** (Text Area): Background information, special circumstances, or additional details
- **Notes** (Text Area): Internal operational notes and observations
- **Relevant Links** (Link): Supporting documentation, photos, or external references
- **Distribution Type** (Single Select): Group as final recipient, Individual-Equal, Individual-Custom
- **Priority** (Single Select): Low, Medium, High, Urgent
- **Needs Spreadsheet** (file): a file structured as a [Needs Spreadsheet](needs_spreadsheet.md). Optional if the user has hand introduced Needs (see next section)

### Mixed Fields
- ** Needs ** (Need): the needs associated with this Need package. Can be inputed by the user or generated from the Needs Spreadsheet.
- **Status** (Calculated Select): Draft, Finalized, Accepted, Declined, Paid, Partially Paid, Cancelled
  - **Automation**: "Paid" when all active child Needs are Paid, "Partially Paid" when some active child Needs are Paid, "Declined" when all active child Needs are Declined
  - **Manual override**: User can change to "Cancelled" for operational exceptions (e.g., recipient group disbanded, circumstances changed)
  - **Override scenarios**: Package-level decisions that override individual Need status aggregation
  - **Finalization lock**: Once "Finalized", most fields become read-only except for manual status overrides

### Strictly Automated Fields
- **Original Total Amount** (Calculated Money): Sum of all [Need](need.md) amounts when package was finalized - calculated once and never changes
- **Current Total Amount** (Calculated Money): Sum of all active [Need](need.md) amounts (excludes cancelled/obsolete Needs)
- **Need Count** (Calculated Number): Total number of individual [Need](need.md)s in this package
- **Unique Recipients** (Calculated Number): Total number of distinct recipients across all Needs
- **Type Summary** (Calculated Text): Breakdown by need type showing number of needs, total amount, and percentage (e.g., "Transport: 25 needs, 125,000 KES (83%); Education: 5 needs, 25,000 KES (17%)")
- **Recipients** (Relationship): Links to [Person](person.md)/[School](school.md)/[Group](group.md) records for intended recipients, imported automatically from the Needs associated with this NeedPackage 

### Buttons/Bulk Actions
- **Finalize Package** (Single Select): "Finalize Now", "None" - Locks Original Total Amount, generates individual Needs, triggers notifications
- **Mark Needs Status** (Single Select): "Mark all Accepted Needs as Paid", "None" - Triggers bulk status update operations, resets to "None" after execution

### System Fields
- **System Messages** (Text Area): Automation log for status calculations, finalization issues, and bulk operation results
  - Example: "2025-09-06 14:30: Status calculation - 45 of 50 Needs marked Paid, status changed to Partially Paid"
  - Example: "2025-09-06 15:45: Finalization - Package locked, 50 individual Needs created, notifications sent"
  - Example: "2025-09-06 16:20: Bulk operation - Mark all Accepted Needs as Paid completed, 48 successful, 2 failed"
- **Created Date** (Date): When this package was created
- **Last Updated** (Date): Most recent modification

## Rules
- NeedPackage can contain multiple recipients and multiple need types within the same package
- All child Needs must use the same currency as the NeedPackage
- Package starts in "Draft" status - fully mutable, Original Total Amount not calculated
- "Finalize Package" button calculates Original Total Amount, generates individual Need records, changes status to "Finalized", and triggers notifications to transaction responsibility team
- After finalization, package becomes mostly immutable - amendments require special workflow or separate amendment package
- Status automatically calculated: "Paid" when all active Needs paid, "Partially Paid" when any active Needs paid
- Current Total Amount excludes Needs with status "Cancelled" or "Obsolete"
- Need Count, Type Summary, and Unique Recipients automatically update when child Needs change
- Distribution Type determines how individual Needs are generated from the package
- Individual child Needs maintain atomic principle: one recipient + one need type + one amount per Need
- Bulk action "Mark all Accepted Needs as Paid" updates allocation statuses for Needs with status "Accepted", which then propagates to Need status recalculation
- For audit purposes, NeedPackages should be marked as obsolete/cancelled rather than deleted to preserve financial history

## Examples
1. **Mixed Request**: "School F New Semester Package" - Contains 30 students needing both school fees (Education type) and uniforms (Infrastructure type). Distribution Type: Individual-Custom. Generates 60 individual Needs: 30 for fees, 30 for uniforms, each with specific recipient and amount.

2. **Single Type Group**: "Holiday Transport Package" - Contains 25 students needing transport home. Distribution Type: Individual-Equal. Generates 25 individual Needs, all Transport type, equal amounts per student, but each with specific recipient for accountability.