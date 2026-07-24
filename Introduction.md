# IBM Business Automation Workflow (BAW) – Overview

## What is IBM BAW?

IBM Business Automation Workflow (BAW) is an enterprise workflow automation platform that combines **Business Process Management (BPM)** and **Case Management** into a single product. It helps organizations automate business processes involving people, documents, business rules, and external systems.

### Key Features
- Business Process Management (BPM)
- Case Management
- Human Workflow
- Business Rules
- Document Management Integration (IBM FileNet)
- REST/SOAP Integration
- Workflow Monitoring & Analytics

---

# IBM BAW Architecture

```text
                    Users
                      │
          Process Portal / Case Client
                      │
               Human Services
                      │
        +--------------------------------+
        |      IBM BAW Server            |
        |--------------------------------|
        | Process Engine                 |
        | Case Management                |
        | Decision Services              |
        | Integration Services           |
        +--------------------------------+
             │         │          │
             │         │          │
      REST APIs    Databases   IBM FileNet
             │                    │
      External Systems       Documents
```

---

# IBM BAW Components

---

# 1. Process Application

## Description
A Process Application is the main container that stores all workflow-related artifacts.

## Contains
- BPDs
- Coaches
- Human Services
- Business Objects
- Services
- Decision Tables
- Variables

## Real-Time Example

**Loan Origination System**

```
LoanOrigination Process App

├── Loan Approval BPD
├── Credit Score Service
├── Loan Coach
├── Decision Table
├── Business Objects
└── REST Integrations
```

---

# 2. Business Process Definition (BPD)

## Description

A Business Process Definition (BPD) represents the workflow of a business process.

## Real-Time Example

### Loan Approval Process

```text
Start

↓

Submit Loan

↓

Credit Bureau API

↓

Calculate FOIR

↓

Decision Service

↓

Auto Approve?
     │
Yes  │ No
│    │
Approve
     │
     ↓
Junior Underwriter

↓

Senior Underwriter

↓

Disbursement

↓

End
```

## Use Cases

- Loan Approval
- Insurance Claim
- Purchase Approval
- Employee Onboarding

---

# 3. Human Service

## Description

Human Services provide screens where users perform tasks.

## Examples

- Loan Application Form
- Insurance Claim Form
- Leave Approval Form

## Real-Time Example

```
Loan Details

Applicant Name

PAN

Salary

Loan Amount

Submit
```

---

# 4. Coach

## Description

A Coach is the user interface (UI) displayed to business users.

## Components

- Text Box
- Date Picker
- Dropdown
- Button
- Table
- File Upload
- Check Box

## Example

```
Loan Application

Name

PAN

Salary

Loan Amount

Tenure

Submit
```

---

# 5. Coach View

## Description

Coach Views are reusable UI components.

## Examples

- PAN Input
- Aadhaar Input
- EMI Calculator
- Address Component

## Real-Time Example

Instead of creating PAN fields on 40 screens,

Create one reusable Coach View.

```
MaskedPANCoachView
```

Use it everywhere.

---

# 6. Business Objects (BO)

## Description

Business Objects store business data.

## Example

```
LoanApplication

Applicant
    Name
    PAN
    Salary

Loan
    Amount
    Tenure

Credit
    Score

Decision
    Status
```

## Real-Time Example

Every loan application has one LoanApplication Business Object.

---

# 7. Variables

## Description

Variables store runtime data.

## Examples

```
tw.local.loan

tw.local.applicant

tw.local.creditScore

tw.local.caseFolder

tw.system.user
```

## Use Cases

- Store API Response
- Store User Input
- Store Decision Result

---

# 8. Integration Service

## Description

Used to communicate with external applications.

## Supports

- REST
- SOAP
- Java
- Database
- FileNet
- Email

## Real-Time Example

```
BAW

↓

Credit Bureau API

↓

Credit Score

↓

Continue Process
```

---

# 9. Decision Service

## Description

Decision Services execute business rules.

## Example

```
Credit Score > 750

FOIR < 50%

↓

Auto Approve
```

Else

```
Route to Underwriter
```

## Use Cases

- Loan Eligibility
- Interest Rate Calculation
- Insurance Premium
- Discount Calculation

---

# 10. Decision Table

## Example

| Credit Score | FOIR | Decision |
|--------------|------|----------|
| >750 | <50% | Auto Approve |
| 700-750 | <60% | Refer |
| <650 | Any | Reject |

---

# 11. System Service

## Description

Runs automatically without user interaction.

## Examples

- Database Insert
- REST Call
- Email
- PDF Generation
- File Upload

## Example

```
Submit Loan

↓

Save Database

↓

Generate Loan Number

↓

Send Email
```

---

# 12. AJAX Service

## Description

Runs in the background without refreshing the page.

## Example

```
User enters PAN

↓

Validate PAN

↓

Fetch Customer Name

↓

Display Automatically
```

---

# 13. Event

## Types

- Message Event
- Timer Event
- Error Event
- Signal Event

## Example

```
Payment Received

↓

Continue Workflow
```

---

# 14. Timer

## Description

Executes after a configured duration.

## Real-Time Example

```
Junior Underwriter

30 Minutes

↓

No Action

↓

Escalate

↓

Senior Underwriter
```

---

# 15. Undercover Agent (UCA)

## Description

Listens for external events and starts or continues a process.

## Example

```
Payment Gateway

↓

Payment Successful

↓

Trigger Loan Disbursement
```

---

# 16. Teams

## Description

Teams define task assignments.

## Example

```
Loan Amount < 20 Lakhs

↓

Junior Underwriter

Loan Amount ≥ 20 Lakhs

↓

Senior Underwriter
```

---

# 17. Process Portal

## Description

Users complete assigned tasks from the Process Portal.

## Example

```
Inbox

Loan Approval

Insurance Claim

Expense Approval

Leave Approval
```

---

# 18. Case

## Description

Cases are used when the workflow is dynamic and unpredictable.

## Use Cases

- Insurance Claim
- Fraud Investigation
- Customer Complaint
- Legal Case

---

# 19. Case Folder

## Description

Stores all documents related to a case.

## Example

```
Loan Folder

PAN Card

Salary Slip

Photo

Aadhaar

Agreement

Sanction Letter
```

---

# 20. IBM FileNet Integration

## Description

BAW integrates with IBM FileNet for document management.

## Example

```
Upload Document

↓

Store in FileNet

↓

Link to Case

↓

Retrieve Anytime
```

---

# 21. Snapshot

## Description

A Snapshot is a version of a Process Application.

## Example

```
Loan Application

Version 1.0

↓

Version 1.1

↓

Version 2.0
```

---

# 22. Workflow Center

## Description

Development environment.

## Responsibilities

- Design
- Develop
- Test
- Snapshot Creation

---

# 23. Workflow Server

## Description

Runtime environment.

## Responsibilities

- Execute Processes
- Run Services
- Handle User Tasks
- Execute Timers

---

# End-to-End Real-Time Loan Approval Example

```text
Customer Applies Loan
        │
        ▼
Loan Officer (Human Service)
        │
        ▼
Save Application (System Service)
        │
        ▼
Credit Bureau API (Integration Service)
        │
        ▼
Calculate FOIR
        │
        ▼
Decision Service
        │
        ├───────────────┐
        │               │
        ▼               ▼
Auto Approve      Underwriter Review
        │               │
        ▼               ▼
Approve         Junior Underwriter
                        │
                        ▼
                 Timer (30 Minutes)
                        │
                        ▼
                 Escalate to Senior UW
                        │
                        ▼
          Upload Documents (FileNet)
                        │
                        ▼
               Loan Disbursement
                        │
                        ▼
                  Email Customer
                        │
                        ▼
                       End
```

---

# Summary Table

| Component | Purpose | Real-Time Example |
|-----------|---------|-------------------|
| Process Application | Stores all artifacts | Loan Origination |
| BPD | Defines workflow | Loan Approval Process |
| Human Service | User interaction | Loan Application Form |
| Coach | User Interface | Loan Entry Screen |
| Coach View | Reusable UI | PAN Masking |
| Business Object | Stores business data | LoanApplication |
| Variables | Runtime data | Credit Score |
| Integration Service | Connect external systems | Credit Bureau API |
| Decision Service | Execute business rules | Loan Approval Rules |
| Decision Table | Rule Matrix | Credit Score vs FOIR |
| System Service | Background execution | Save to Database |
| AJAX Service | Background validation | PAN Validation |
| Timer | Escalation | Underwriter Timeout |
| UCA | Event listener | Payment Success |
| Team | Task Assignment | Junior/Senior Underwriter |
| Process Portal | User Inbox | Loan Tasks |
| Case | Dynamic workflows | Insurance Claims |
| Case Folder | Document storage | Loan Documents |
| FileNet Integration | Document management | Store Loan Files |
| Snapshot | Versioning | Release v1.0 |
| Workflow Center | Development | Build & Test |
| Workflow Server | Runtime | Execute Workflows |
