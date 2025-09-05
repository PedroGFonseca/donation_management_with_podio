# NeedPackage

## What It Is
A group-level funding request that contains multiple individual Need records, providing clean UI management while maintaining detailed individual accountability.

## Purpose
To organize related funding requests into manageable units that can mix multiple recipients and multiple need types, while automatically generating precise individual Need records for perfect accountability.

## Why it is needed 
NeedPackage solves the granularity dilemma by enabling clean UI organization without sacrificing individual tracking. It allows natural bundling of requests (like "School F new semester needs") while ensuring every recipient and amount is precisely tracked through individual Need records.

## Key Relationships
- Parent: Foundation donation system, optionally generated from NeedTemplate
- Children: Individual Need records (one per recipient per need type)
- Links to: NeedTemplate (if generated from template), Allocation records (through child Needs)

## Fields

### Core Fields
- **Total Amount** (Calculated Money): Sum of all individual Need amounts within this package

### Administrative Fields  
- **Status** (Calculated Select): New, Accepted, Declined, Paid, Partially Paid, Cancelled
- **Distribution Type** (Single Select): Group-Only, Individual-Equal, Individual-Custom
- **Priority** (Single Select): Low, Medium, High, Urgent

### Optional Fields
- **Title** (Text): Descriptive name for this group request
- **Description** (Text Area): Overall description of the request and context
- **Currency** (Single Select): Currency for all amounts in this package (defaults to foundation's default currency)
- **Request Date** (Date): When this request was submitted
- **Deadline** (Date): When funding is needed by
- **Recipients** (Text Area): List or description of intended recipients (used for Need generation)
- **Context** (Text Area): Background information, special circumstances, or additional details
- **Notes** (Text Area): Internal operational notes and observations
- **Relevant Links** (Link): Supporting documentation, photos, or external references

### System Fields
- **Template Reference** (Relationship): Links to NeedTemplate if generated from template
- **Created Date** (Date): When this package was created
- **Last Updated** (Date): Most recent modification
- **Need Count** (Calculated Number): Total number of individual Needs in this package
- **Type Summary** (Calculated Text): Auto-generated breakdown of need types (e.g., "Transport: 25 needs (83%), Education: 5 needs (17%)")

## Rules
- NeedPackage can contain multiple recipients and multiple need types within the same package
- All child Needs must use the same currency as the NeedPackage
- Status automatically calculated: "Paid" when all Needs paid, "Partially Paid" when any Needs paid
- Total Amount, Need Count, and Type Summary automatically update when child Needs change
- Distribution Type determines how individual Needs are generated from the package
- Individual child Needs maintain atomic principle: one recipient + one need type + one amount per Need
- Unique Recipients count handles cases where one person appears in multiple Needs (e.g., John needs both fees and uniforms)
- For audit purposes, NeedPackages should be marked as obsolete/cancelled rather than deleted to preserve financial history

## Examples
1. **Mixed Request**: "School F New Semester Package" - Contains 30 students needing both school fees (Education type) and uniforms (Infrastructure type). Distribution Type: Individual-Custom. Generates 60 individual Needs: 30 for fees, 30 for uniforms, each with specific recipient and amount.

2. **Single Type Group**: "Holiday Transport Package" - Contains 25 students needing transport home. Distribution Type: Individual-Equal. Generates 25 individual Needs, all Transport type, equal amounts per student, but each with specific recipient for accountability.