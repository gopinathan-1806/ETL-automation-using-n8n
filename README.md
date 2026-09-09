# AI-Powered Student Enrollment ETL Pipeline with n8n

An end-to-end ETL automation built with **n8n and an LLM** to clean
messy student enrollment data and route each record into the correct
output based on **city** and **email validity**.

## Problem Statement

Social Eagle AI Academy receives raw student enrollment data every week
from Google Forms. The incoming data is messy and can contain
inconsistent names, invalid emails, missing phone numbers, inconsistent
course names, different fee-payment values, inconsistent cities, and
different date formats.

The goal is to build an automated ETL pipeline that:

1.  Extracts raw CSV data.
2.  Processes every student row independently.
3.  Uses an LLM to clean and standardize the data.
4.  Applies city and email conditions.
5.  Generates four clean output CSV files.

## Workflow Architecture

``` text
Raw CSV
   ↓
Extract from File
   ↓
Loop Over Items
   ↓
Basic LLM Chain
   ↓
Code in JavaScript
   ↓
IF — Chennai / Other Cities
   ├── Chennai → IF — Email
   │                ├── Valid   → chennai_valid_email.csv
   │                └── Invalid → chennai_invalid_email.csv
   │
   └── Other Cities → IF — Email
                        ├── Valid   → other_valid_email.csv
                        └── Invalid → other_invalid_email.csv
```

The **Loop Over Items** node is used so each student record is processed
individually by the AI cleaning step.

## Tools Used

-   **n8n** --- workflow automation and orchestration
-   **LLM** --- AI-based data cleaning and normalization
-   **CSV** --- input and output format
-   **JavaScript** --- additional data transformation
-   **Google Forms** --- source system described in the problem
    statement

## Input Dataset

The workflow uses:

``` text
student_enrollment_raw.csv
```

The dataset contains **50 messy student enrollment records**.

Example raw record:

``` text
Student_ID: STU1001
Name: manoJ k
Email: manojk@gmail
Phone: 9876543210
Course: python
Fee_Paid: yes
City: Chennai
Enrolled_Date: 23-05-2025
```

Example cleaned record:

``` json
{
  "Student ID": "STU1001",
  "Name": "Manoj K",
  "Email": "INVALID_EMAIL",
  "Phone": "9876543210",
  "Course": "Python",
  "Fee Paid": true,
  "City": "Chennai",
  "Enrolled Date": "2025-05-23"
}
```

## AI Data Cleaning

Each row is sent to the LLM through the **Basic LLM Chain**.

  Field           Cleaning Rule
  --------------- --------------------------------------------------------
  Name            Title Case
  Email           Invalid/incomplete email → `INVALID_EMAIL`
  Phone           Less than 10 digits → `MISSING`
  Course          Normalize to Python, Machine Learning, or Data Science
  Fee Paid        `yes/YES` → `true`; `no/NO` → `false`
  City            Title Case; empty → `UNKNOWN`
  Enrolled Date   Convert to `YYYY-MM-DD`

The LLM is used for data cleaning and normalization, while deterministic
IF nodes handle the final business routing.

## Filtering Logic

### Condition 1 --- Location

``` text
City = Chennai
    → Chennai branch

City != Chennai
    → Other Cities branch
```

### Condition 2 --- Email

``` text
Email != INVALID_EMAIL
    → Valid Email

Email = INVALID_EMAIL
    → Invalid Email
```

Combining both conditions creates four output groups:

  Location       Email     Output
  -------------- --------- -----------------------------
  Chennai        Valid     `chennai_valid_email.csv`
  Chennai        Invalid   `chennai_invalid_email.csv`
  Other Cities   Valid     `other_valid_email.csv`
  Other Cities   Invalid   `other_invalid_email.csv`

## n8n Workflow Nodes

### 1. On Form Submission

Starts the workflow.

### 2. Extract from File

Reads the raw CSV and converts its contents into individual data items.

### 3. Loop Over Items

Processes the 50 records one at a time.

### 4. Basic LLM Chain

Sends each student row to the LLM and receives cleaned structured data.

### 5. OpenAI Chat Model

Provides the language model used by the Basic LLM Chain.

### 6. Code in JavaScript

Performs additional transformation/handling of the cleaned AI output.

### 7. IF --- Location Filter

Routes records into Chennai and Other Cities branches.

### 8. IF --- Email Filter

Routes each city branch into Valid Email and Invalid Email.

### 9. Convert to File

Converts each filtered set of records into CSV output.

## Example Routing

For:

``` json
{
  "Student ID": "STU1004",
  "Name": "Anitha R",
  "Email": "anitha@gmail.com",
  "City": "Chennai"
}
```

The workflow evaluates:

``` text
City = Chennai
    ↓ TRUE
Email = INVALID_EMAIL?
    ↓ FALSE
Valid Email
    ↓
chennai_valid_email.csv
```

For:

``` json
{
  "Student ID": "STU1001",
  "Name": "Manoj K",
  "Email": "INVALID_EMAIL",
  "City": "Chennai"
}
```

The workflow evaluates:

``` text
City = Chennai
    ↓ TRUE
Email = INVALID_EMAIL
    ↓ TRUE
Invalid Email
    ↓
chennai_invalid_email.csv
```

## Key Learning

This project demonstrates a practical pattern for combining **AI with
deterministic data engineering workflows**.

``` text
Messy Data
    ↓
LLM Cleaning / Normalization
    ↓
Structured Data
    ↓
Deterministic IF Conditions
    ↓
Business Routing
    ↓
Clean Output Files
```

The LLM handles tasks where language understanding is useful. The
workflow handles explicit business rules using deterministic conditions.

## Challenges Solved

-   Processing every CSV row independently using Loop Over Items
-   Standardizing inconsistent student data with an LLM
-   Converting incomplete emails to `INVALID_EMAIL`
-   Combining city and email conditions
-   Routing records into four output groups
-   Converting filtered records back into CSV files

## Workflow Screenshot

Add the exported workflow screenshot to the repository as:

``` text
workflow.png
```

Then it will appear here:

![Completed n8n workflow](workflow.png)

## Project Outcome

Successfully built an end-to-end AI-assisted ETL pipeline that:

-   Extracts 50 raw student records from CSV
-   Processes records individually using an LLM
-   Cleans and standardizes inconsistent data
-   Separates Chennai and Other Cities
-   Separates Valid and Invalid Email records
-   Produces four clean CSV outputs

## Future Improvements

-   Replace CSV input with a Google Sheets / Google Forms integration
-   Write final records directly to Google Sheets
-   Add post-LLM data-quality validation
-   Add error handling and retry logic
-   Add execution monitoring and notifications
-   Store workflow execution logs
-   Use deterministic validation for fields that should not depend on an
    LLM
-   Add more business rules as the dataset grows

## Project Flow Summary

``` text
EXTRACT
  ↓
Raw Student CSV
  ↓
TRANSFORM
  ↓
Loop → LLM Data Cleaning → JavaScript
  ↓
FILTER
  ↓
City Condition
  ↓
Email Condition
  ↓
LOAD
  ↓
chennai_valid_email.csv
chennai_invalid_email.csv
other_valid_email.csv
other_invalid_email.csv
```

**Result: A complete AI-powered ETL automation pipeline built with
n8n.**


<img width="1496" height="770" alt="Screenshot 2026-09-09 at 3 47 17 PM" src="https://github.com/user-attachments/assets/e090e469-a12a-4ad8-88dd-cf6cb99f9cdf" />

