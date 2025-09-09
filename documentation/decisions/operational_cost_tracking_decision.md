# Operational Cost Tracking - Foundation System Design Decision

*Decision Document - September 2025*

## Problem Statement

The foundation incurs various operational costs related to money management that don't fit cleanly into individual Transaction records but need to be tracked for accurate financial reporting and donor accountability. These include:

- **Currency conversion costs**: Batch conversions where multiple transactions are converted together with mixed fixed/variable cost structures
- **Account loading costs**: Fees for transferring money into TransferWise or other payment accounts
- **Banking maintenance fees**: Monthly/annual account fees
- **Exchange rate timing losses**: Costs from currency fluctuation timing
- **Other operational overhead**: Administrative costs related to money management

## The Attribution Problem

### **Why Not Include in Transactions**
Individual transactions cannot accurately capture these costs because:

**Batch conversion example:**
- Load €10,000 to TransferWise account (loading fee: €15)
- Convert €10,000 to KES over 3 weeks as transactions are needed
- Conversion spread: 2% fixed + €5 per conversion
- Results in 8 individual transactions to Kenya
- **Problem**: How to allocate the €15 loading fee + conversion costs across 8 transactions that happened at different times?

**Mixed cost structures:**
- Fixed components (loading fees, conversion minimums)
- Variable components (percentage spreads, per-transaction fees)
- Timing dependencies (exchange rate changes)
- Batch operations (bulk conversions)

## Decision: Separate Operational Cost Entity

### **Core Principle**
Create a separate "OperationalCost" entity to track foundation overhead costs that cannot be cleanly attributed to individual transactions but affect overall funding availability.

### **OperationalCost Entity Design**

#### **Purpose**
Track foundation overhead costs related to money management that reduce overall funding capacity but cannot be attributed to specific transactions.

#### **Key Relationships**
- **References**: Foundation Account (source account), Time Period (when incurred)
- **Referenced by**: None (overhead entity)

#### **Fields**

**User Input Fields:**
- **Title** (Text): Descriptive name for this cost
- **Cost Type** (Single Select): Currency Conversion, Account Loading, Banking Fees, Exchange Rate Loss, Administrative, Other
- **Amount** (Money): Cost amount
- **Currency** (Single Select): Currency of the cost
- **Foundation Account** (Relationship): Which foundation account incurred this cost
- **Date Incurred** (Date): When the cost was incurred
- **Time Period** (Text): Month/quarter this cost applies to (e.g., "March 2025")
- **Description** (Text Area): Details about what generated this cost
- **Relevant Links** (Link): Bank statements, TransferWise fee reports
- **Attach Files** (File): Supporting documentation

**Mixed Fields:**
- **Status** (Single Select): Pending, Confirmed, Disputed
  - **Automation**: "Confirmed" when documentation uploaded
  - **Manual override**: "Disputed" for billing issues

**Strictly Automated Fields:**
- **Impact on Available Funding** (Money): How this cost reduces overall funding capacity

**System Fields:**
- **System Messages** (Text Area): Cost calculation updates, period allocation changes
- **Created Date**, **Last Updated** (Auto)

### **Integration with FundingBatch System**

#### **Funding Capacity Calculation**
```
Foundation Total Capacity = 
  Sum(Successful Transaction Net Amounts) 
  - Sum(Confirmed Operational Costs)
  - Reserved Emergency Fund
```

#### **Cost Attribution Methods**

**Option 1: Time-Based Allocation**
- Allocate operational costs to FundingBatches based on creation date
- Costs reduce available credit for batches created in that period

**Option 2: Proportional Allocation**
- Allocate operational costs proportionally across all active FundingBatches
- Costs distributed based on FundingBatch size

**Option 3: Foundation Overhead Pool**
- Track operational costs separately from FundingBatch system
- Report as foundation overhead in donor communications
- Don't reduce individual FundingBatch capacity

**Recommended Approach: Option 3** - Foundation overhead tracking without direct FundingBatch impact for simplicity and clarity.

### **Real-World Examples**

#### **Example 1: Batch Currency Conversion**
```
OperationalCost: "March 2025 EUR-KES Conversion"
- Cost Type: Currency Conversion
- Amount: €247.50
- Description: "Converted €10,000 to KES over 3 weeks. Fixed spread: €200, Variable: €47.50"
- Time Period: "March 2025"
- Foundation Account: "TransferWise EUR Account"
- Impact: Reduces overall foundation capacity by €247.50
```

#### **Example 2: Account Loading**
```
OperationalCost: "TransferWise Account Top-up Fee"
- Cost Type: Account Loading  
- Amount: €15.00
- Description: "Fee for transferring €5,000 from main bank to TransferWise"
- Foundation Account: "Main EUR Account"
- Time Period: "February 2025"
```

#### **Example 3: Banking Maintenance**
```
OperationalCost: "Monthly Banking Fees"
- Cost Type: Banking Fees
- Amount: €25.00
- Description: "Monthly maintenance fee for TransferWise business account"
- Time Period: "March 2025"
- Foundation Account: "TransferWise EUR Account"
```

## Benefits of This Approach

### **Clean Transaction Records**
- Transactions focus purely on money movement to Kenya
- No messy cost allocation across unrelated transactions
- Clear distinction between transfer costs and conversion overhead

### **Accurate Foundation Reporting**
- Complete picture of money management costs
- Proper overhead tracking for donor reporting
- ANBI compliance with complete cost documentation

### **Operational Insight**
- Visibility into conversion timing efficiency
- Cost optimization opportunities (batch vs individual conversions)
- Account management cost analysis

### **System Simplicity**
- No complex cost allocation algorithms in Transaction entity
- Clear separation of concerns
- Easier to modify cost tracking without affecting core transaction logic

## Alternative Approaches Rejected

### **Transaction-Level Cost Allocation**
**Rejected**: Complex attribution problems, arbitrary cost splitting, timing issues

### **FundingBatch-Level Cost Tracking**
**Rejected**: Costs often span multiple FundingBatches, timing mismatches

### **Ignore Operational Costs**
**Rejected**: Incomplete financial picture, missed optimization opportunities

## Implementation Priorities

### **Phase 1: Basic Cost Tracking**
- OperationalCost entity with core fields
- Manual cost entry and documentation
- Simple reporting for foundation overhead

### **Phase 2: Integration with Funding Reporting**
- Foundation capacity calculations
- Donor reporting integration
- Cost trend analysis

### **Phase 3: Advanced Analytics**
- Cost optimization recommendations
- Conversion timing analysis
- Account efficiency metrics

## Success Criteria

This approach succeeds if:
- All foundation money management costs are captured accurately
- Transaction records remain clean and focused
- Foundation overhead is properly reported to donors
- Cost optimization opportunities are identified
- ANBI compliance requirements are met with complete documentation

---

*This decision establishes a clean separation between direct transaction costs and foundation operational overhead, enabling accurate financial reporting while maintaining system simplicity.*
