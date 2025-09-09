# Transaction

## What It Is
A record of money transfer from a foundation account to Kenya, capturing the banking/logistics details of how funds moved from the foundation to local intermediaries or recipients.

## Purpose
To maintain complete records of actual money movements while feeding the FundingBatch system with available funding. Transactions focus purely on the banking mechanics while leaving recipient assignment to the Allocation system.

## Why it is needed 
Transaction records provide the foundation for financial accountability by tracking exactly how much money was sent, what it cost to send, and who received it locally. This enables accurate FundingBatch funding levels while maintaining complete audit trails for ANBI compliance.

## Key Relationships
- **References**: Foundation Account (source account field), FundingBatch (fundingbatch field), Local Intermediary (intermediary field - optional)
- **Referenced by**: None (funding source entity)

## Fields

### User Input Fields
- **Title** (Text): Descriptive name for this transaction
- **Foundation Account** (Relationship): Which foundation account sent the money (links to foundation accounts app)
- **FundingBatch** (Relationship): Which FundingBatch this transaction funds
- **Amount Sent** (Money): Amount initiated from foundation account
- **Currency Sent** (Single Select): Original currency (EUR, USD, KES, etc.)
- **Amount Received** (Money): Amount that arrived at destination (after transfer fees)
- **Currency Received** (Single Select): Destination currency (usually KES)
- **Date Sent** (Date): When transaction was initiated
- **Date Received** (Date): When money arrived at destination (may be same as Date Sent for cash)
- **Local Intermediary** (Person/Local Partner): Who physically receives the money in Kenya (optional - money might go directly to final recipient)
- **Transfer Service** (Text): TransferWise, M-Pesa Direct, Bank Transfer, Cash, etc.
- **Description** (Text Area): Transaction context and operational notes
- **Relevant Links** (Link): Banking confirmations, TransferWise transaction URLs
- **Attach Files** (File): Banking documents, receipts, screenshots

### Mixed Fields
- **Status** (Single Select): Incomplete, Complete, Failed, Cancelled
  - **Automation**: "Complete" when all required fields filled and transaction confirmed
  - **Manual override**: User can mark "Failed" or "Cancelled" for banking issues
  - **Override scenarios**: Banking failures, recipient unavailable, transaction disputes
  - **Constraint**: Cannot mark "Complete" unless FundingBatch assigned and amounts confirmed

### Strictly Automated Fields
- **Transfer Fee** (Money): Fees charged by transfer service (calculated from Amount Sent - Amount Received when same currency)
- **Net Contribution to FundingBatch** (Money): Amount that becomes available for allocation (equals Amount Received)

### System Fields
- **Creation Method** (Single Select): "API Import", "Manual Entry", "Template"
- **TransferWise ID** (Text): Reference ID for TransferWise transactions (empty for non-TransferWise)
- **Paid By** (Person/Organization): Who funded this transaction (auto-populated from Foundation Account)
- **System Messages** (Text Area): Transaction processing updates, validation issues, banking confirmations
  - Example: "2025-09-15 16:45: Auto-imported from TransferWise API - TW-98765"
  - Example: "2025-09-15 17:20: User completed business fields - Status changed to Complete"
  - Example: "2025-09-15 17:25: FundingBatch credit updated - Added 65,750 KES to Student Transport March 2025"
- **Created Date** (Date): When this transaction record was created
- **Last Updated** (Date): Most recent modification

## Rules
- Transaction must specify exactly one Foundation Account (source of money)
- Transaction must specify exactly one FundingBatch (destination for available funding)
- Amount Received determines how much credit is added to the FundingBatch
- Status "Complete" requires: FundingBatch assigned, amounts confirmed, transfer confirmed
- Local Intermediary is optional - some transactions go directly to final recipients
- Transfer Fee automatically calculated for same-currency transactions (Amount Sent - Amount Received)
- Currency conversion costs are NOT tracked at transaction level (see [Operational Cost Tracking](../decisions/operational_cost_tracking_decision.md))
- TransferWise transactions may be auto-imported (see [Transaction Import Automation](../automation/transaction_import.md))
- For audit purposes, Transactions should be marked as obsolete/cancelled rather than deleted to preserve financial history
- One Transaction can only fund one FundingBatch, but one FundingBatch can receive multiple Transactions
- Money becomes fungible within FundingBatch - specific transaction source doesn't matter for allocations

## Examples

### Example 1: TransferWise Auto-Import
**Auto-imported fields:**
- Title: "TW-98765 - EUR to Kenya"
- Foundation Account: "TransferWise EUR Account"
- Amount Sent: €500.00
- Amount Received: 65,750 KES
- Transfer Service: "TransferWise"
- Transfer Fee: €4.50
- Status: "Incomplete"

**User completes:**
- FundingBatch: "Student Transport March 2025"
- Local Intermediary: "Teacher Mary Wanjiku" 
- Description: "Transport funding for 15 students via Teacher Mary"
- Title: "Student Transport - Teacher Mary (TW-98765)" (customized)
- Status: "Complete" (auto-updated)

### Example 2: Manual Cash Transaction
**User creates manually:**
- Title: "Emergency Medical - Local Clinic"
- Foundation Account: "Petty Cash KES"
- FundingBatch: "Emergency Response March 2025"
- Amount Sent: 15,000 KES
- Amount Received: 15,000 KES
- Currency Sent: KES
- Currency Received: KES
- Date Sent: 2025-09-15
- Date Received: 2025-09-15
- Local Intermediary: "Dr. Kiprotich"
- Transfer Service: "Cash"
- Transfer Fee: 0 KES
- Description: "Emergency medical payment for student Alice"
- Status: "Complete"

### Example 3: M-Pesa Direct Transfer
**User creates manually:**
- Title: "Teacher Salary - John Mwangi"
- Foundation Account: "M-Pesa Business Account"
- FundingBatch: "Staff Salaries March 2025"
- Amount Sent: 25,000 KES
- Amount Received: 24,750 KES
- Currency Sent: KES
- Currency Received: KES
- Transfer Service: "M-Pesa Direct"
- Transfer Fee: 250 KES
- Local Intermediary: "John Mwangi (Teacher)"
- Description: "Monthly salary payment"
- Status: "Complete"