# IBM BAW – Process of Data Storing into Database (DB)

## Overview

In IBM Business Automation Workflow (BAW), data entered by users is **not immediately stored in your application's business database**. Instead, it flows through several layers before being persisted.

There are two types of data storage in BAW:

1. **Process Data (Internal BAW Database)** – Stores workflow execution information.
2. **Business Data (Application Database)** – Stores business-specific information such as loan applications, customers, policies, etc.

---

# Data Flow Architecture

```text
User
 │
 ▼
Coach (UI)
 │
 ▼
Business Object (tw.local)
 │
 ▼
Human Service
 │
 ▼
Service Flow
 │
 ▼
Integration Service / Java Service / SQL Service
 │
 ▼
Database (Oracle / SQL Server / DB2 / PostgreSQL)
```

---

# Step-by-Step Process

## Step 1: User Opens the Coach

The user opens a Coach (UI) and enters data.

Example:

```
Applicant Name : John

PAN : ABCDE1234F

Salary : 80000

Loan Amount : 1000000
```

At this stage:

- Data exists only in the browser.
- Nothing is stored in the database.

---

## Step 2: Data is Bound to Business Objects

Every Coach control is bound to a Business Object.

Example:

```
Applicant Name
        │
        ▼
tw.local.LoanApplication.applicant.name

PAN
        │
        ▼
tw.local.LoanApplication.applicant.pan

Salary
        │
        ▼
tw.local.LoanApplication.applicant.monthlyIncome
```

Now the data is stored in memory (runtime variables), not yet in the database.

Example:

```javascript
tw.local.LoanApplication.applicant.name = "John";
tw.local.LoanApplication.applicant.pan = "ABCDE1234F";
tw.local.LoanApplication.loan.amount = 1000000;
```

---

## Step 3: User Clicks Submit

```
Submit Button

↓

Human Service Completes

↓

Moves to Next Activity
```

BAW transfers all `tw.local` variables to the next activity.

---

## Step 4: Service Flow Executes

After submission, a Service Flow is invoked.

Example:

```
Submit Loan

↓

Save Loan Details

↓

Call Credit Bureau API

↓

Decision Service
```

The **Save Loan Details** service is responsible for persisting business data.

---

## Step 5: Database Service / Java Service

The service prepares the SQL statement.

Example:

```sql
INSERT INTO LOAN_APPLICATION
(
    APPLICANT_NAME,
    PAN,
    MONTHLY_INCOME,
    LOAN_AMOUNT
)
VALUES
(
    ?,
    ?,
    ?,
    ?
)
```

The placeholders (`?`) are mapped to BAW variables.

---

## Step 6: Variable Mapping

| Database Column | BAW Variable |
|-----------------|-------------|
| APPLICANT_NAME | tw.local.LoanApplication.applicant.name |
| PAN | tw.local.LoanApplication.applicant.pan |
| MONTHLY_INCOME | tw.local.LoanApplication.applicant.monthlyIncome |
| LOAN_AMOUNT | tw.local.LoanApplication.loan.amount |

Example:

```
Applicant Name

↓

tw.local.LoanApplication.applicant.name

↓

SQL Parameter

↓

Database Column
```

---

## Step 7: Database Execution

The SQL statement is executed.

```sql
INSERT INTO LOAN_APPLICATION
VALUES
(
'John',
'ABCDE1234F',
80000,
1000000
)
```

The database stores the record.

Example:

| ID | NAME | PAN | SALARY | LOAN |
|----|------|-----|---------|------|
|101|John|ABCDE1234F|80000|1000000|

---

## Step 8: Response to BAW

The database returns success.

Example:

```text
Rows Inserted = 1
```

The Service Flow continues to the next step.

---

# Complete Flow Diagram

```text
User

↓

Coach

↓

Business Object

↓

Human Service

↓

Service Flow

↓

Database Service

↓

SQL Query

↓

Database

↓

Success Response

↓

Next Activity
```

---

# Example – Loan Approval Process

```text
Customer Opens Loan Form

↓

Enter Details

↓

Click Submit

↓

LoanApplication BO Populated

↓

Save Loan Service

↓

INSERT INTO LOAN_APPLICATION

↓

Database

↓

Generate Application ID

↓

Call Credit Bureau API

↓

Decision Service

↓

Underwriter

↓

Loan Approved
```

---

# Where is Data Stored?

## 1. Runtime Memory (Temporary)

During process execution:

```
tw.local.*

tw.system.*

tw.object.*
```

Examples:

```javascript
tw.local.loanApplication
tw.local.creditScore
tw.local.applicant
```

These variables exist only while the process is running.

---

## 2. BAW Internal Database

IBM BAW automatically stores process-related information such as:

- Process Instance
- Task Information
- Current Activity
- Variables (depending on configuration)
- Audit Logs
- Execution History

Example:

```
Process ID

Current Step

Assigned User

Task Status

Started Time

Completed Time
```

This data is managed by BAW itself.

---

## 3. Business Database

Business data is stored in your application database.

Example Table:

```sql
LOAN_APPLICATION
```

| Column |
|---------|
| APPLICATION_ID |
| APPLICANT_NAME |
| PAN |
| SALARY |
| LOAN_AMOUNT |
| CREDIT_SCORE |
| STATUS |

This data is used by your business application.

---

# Real-Time Banking Example

```text
Customer Applies Loan

↓

Loan Officer enters data

↓

Coach

↓

LoanApplication BO

↓

Submit

↓

Save Loan Service

↓

INSERT INTO LOAN_APPLICATION

↓

Database

↓

Application ID Generated

↓

Credit Bureau API

↓

Decision Service

↓

Auto Approve or Underwriter

↓

Disbursement

↓

End
```

---

# Best Practices

- Use **Business Objects** to hold user input.
- Validate data in the Coach before submission.
- Use **parameterized SQL queries** to prevent SQL injection.
- Separate business data from workflow metadata.
- Handle database errors using exception flows.
- Commit transactions only after successful validation and processing.

---

# Interview Answer

**Q: Explain how data is stored into a database in IBM BAW.**

**Answer:**

> In IBM BAW, users enter data through a Coach, where each UI control is bound to a Business Object (`tw.local`). The data is initially stored in runtime variables. After the user submits the form, a Human Service or Service Flow invokes an Integration, SQL, or Java Service. This service maps the Business Object properties to SQL parameters and executes an `INSERT` or `UPDATE` statement against the business database. Upon successful execution, the database stores the business data, and the workflow continues. Separately, IBM BAW stores process execution details such as task status, process instance, and audit information in its internal workflow database.

---

# Code 

 try{

 log.info(" >>>>> PK_LOAN_APPLICATION Insertion Service Invoked... <<<<<<< "); 

if(tw.local.LoanApplication_BO == null) {

log.info(" ==== PK_LoanApplication_BO is Null ===== ");

 tw.local.LoanApplication_BO = new tw.object.LoanApplication();
 
 //  Initialize nested sub-objects BEFORE setting their properties 
 
  tw.local.LoanApplication_BO.applicant = new tw.object.Applicant();
  
  tw.local.LoanApplication_BO.loan = new tw.object.Loan();
  
  tw.local.LoanApplication_BO.credit = new tw.object.Credit();
  
  tw.local.LoanApplication_BO.decision = new tw.object.Decision();
  
} 

log.info("PK_ApplicationId : " + tw.local.LoanApplication_BO.applicationId);

log.info("PK_ApplicantName : " + tw.local.LoanApplication_BO.applicant.name);

log.info("PK_PAN : " + tw.local.LoanApplication_BO.applicant.pan);

log.info("PK_MonthlyIncome : " + tw.local.LoanApplication_BO.applicant.monthlyIncome);

 log.info("PK_ExistingEMI : " + tw.local.LoanApplication_BO.applicant.existingEMI);
 
 log.info("PK_ProductCode : " + tw.local.LoanApplication_BO.loan.productCode);
 
 log.info("PK_RequestedAmount : " + tw.local.LoanApplication_BO.loan.requestedAmount);
 
log.info("PK_TenureMonths : " + tw.local.LoanApplication_BO.loan.tenureMonths);

log.info("PK_InterestRate : " + tw.local.LoanApplication_BO.loan.interestRate);

log.info("PK_BureauScore : " + tw.local.LoanApplication_BO.credit.bureauScore);

log.info("PK_FOIR : " + tw.local.LoanApplication_BO.credit.FOIR);

tw.local.sqlStatement = new tw.object.SQLStatement(); 

tw.local.sqlStatement.sql = "INSERT INTO [bawdbschema].[PK_LOAN_APPLICATION_DATA]" +
                            "([APPLICATION_ID],[APPLICANT_NAME],[PAN],[MONTHLY_INCOME],[EXISTING_EMI]," +
                            "[PRODUCT_CODE],[REQUESTED_AMOUNT],[TENURE_MONTHS],[INTEREST_RATE]," + 
                            "[BUREAU_SCORE],[FOIR])" + 
                            " VALUES(?,?,?,?,?,?,?,?,?,?,?)";
                            log.info("PK_SQL : " + tw.local.sqlStatement.sql); 
                      
        
   tw.local.sqlStatement.parameters = []; 
   
   tw.local.sqlStatement.parameters[0] = {}; 
   
   tw.local.sqlStatement.parameters[0].value = tw.local.LoanApplication_BO.applicationId;
   
   tw.local.sqlStatement.parameters[0].type = "VARCHAR";
   
   tw.local.sqlStatement.parameters[1] = {};
   
   tw.local.sqlStatement.parameters[1].value = tw.local.LoanApplication_BO.applicant.name;
   
   tw.local.sqlStatement.parameters[1].type = "VARCHAR";

   
   
   tw.local.sqlStatement.parameters[2] = {};
   
   tw.local.sqlStatement.parameters[2].value = tw.local.LoanApplication_BO.applicant.pan; 
   
   tw.local.sqlStatement.parameters[2].type = "VARCHAR";
   

   
   tw.local.sqlStatement.parameters[3] = {};
   
   tw.local.sqlStatement.parameters[3].value = tw.local.LoanApplication_BO.applicant.monthlyIncome;
   
   tw.local.sqlStatement.parameters[3].type = "DECIMAL"; 
   

   
   tw.local.sqlStatement.parameters[4] = {};
   
   tw.local.sqlStatement.parameters[4].value = tw.local.LoanApplication_BO.applicant.existingEMI; 
   
   tw.local.sqlStatement.parameters[4].type = "DECIMAL"; 
   
   
   tw.local.sqlStatement.parameters[5] = {};
   
   tw.local.sqlStatement.parameters[5].value = tw.local.LoanApplication_BO.loan.productCode;
   
   tw.local.sqlStatement.parameters[5].type = "VARCHAR"; 

   
   
   tw.local.sqlStatement.parameters[6] = {};
   
   tw.local.sqlStatement.parameters[6].value = tw.local.LoanApplication_BO.loan.requestedAmount;
   
   tw.local.sqlStatement.parameters[6].type = "DECIMAL";

   
   
   tw.local.sqlStatement.parameters[7] = {};
   
   tw.local.sqlStatement.parameters[7].value = tw.local.LoanApplication_BO.loan.tenureMonths;
   
   tw.local.sqlStatement.parameters[7].type = "INTEGER";

   
   
   tw.local.sqlStatement.parameters[8] = {}; 
   
   tw.local.sqlStatement.parameters[8].value = tw.local.LoanApplication_BO.loan.interestRate; 
   
   tw.local.sqlStatement.parameters[8].type = "DECIMAL"; 

   
   
   tw.local.sqlStatement.parameters[9] = {};
   
   tw.local.sqlStatement.parameters[9].value = tw.local.LoanApplication_BO.credit.bureauScore;
   
   tw.local.sqlStatement.parameters[9].type = "INTEGER";

   
   
   tw.local.sqlStatement.parameters[10] = {};
   
   tw.local.sqlStatement.parameters[10].value = tw.local.LoanApplication_BO.credit.FOIR;
   
   tw.local.sqlStatement.parameters[10].type = "DECIMAL";


   
   log.info("PK_SQL Statement prepared successfully......");
   
   
   } catch(e) { 
   
         log.error("===== PK_LoanApplication InsertData -- ERROR =====");
         
        log.error("ERROR: " + e.message); 
         throw e;
                
   } 

