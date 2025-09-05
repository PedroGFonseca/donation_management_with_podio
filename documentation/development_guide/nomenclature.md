# Nomenclature Rules

Actor Terminology
- "User": Human operating the Podio system
- "Automation": System-driven actions (webhooks, scripts, calculations)

Podio Concept Capitalization
- Capitalize: When referring to specific Podio concepts (Need, Allocation, FundingBatch, NeedPackage, Transaction)
- Lowercase: When using terms informally in regular language ("Jon needs food")

Example Structure
- "Example 1", "Example 2": Clear separation between different scenarios
- Numbered steps: Use step-by-step format for longer examples rather than long sentences

Naming Conventions
- Use realistic names: Instead of "FB2" use "StudentNutritionMarch2025"
- Allocation naming pattern: "<FundingBatch_name>_<Need_name>" (e.g., "StudentNutritionMarch2025_JonFoodNeed")

Automation Specificity
- Be explicit about automation actions: Instead of "Automation calculates Jon's Need as Paid" use "Automation calculates that sum of Allocations (400,000 KES) equals Jon's Need amount and switches the Status field in Jon's Need record to 'Paid'"

Action Accuracy
- Describe actual mechanisms: Instead of "User triggers bulk allocation" use "User changes Bulk Operations field from 'None' to 'Auto-allocate'"