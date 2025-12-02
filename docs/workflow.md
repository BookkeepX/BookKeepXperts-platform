# BookKeepXperts – Golden-Path Workflow

This document describes the **Golden Path** workflow for the BookKeepXperts platform.

The Golden Path is the **simplest, end-to-end happy flow** that represents how the system should behave for a typical small business client when everything works correctly.

It covers the journey from:

> Client onboarding → Bank transactions → AI categorization → QuickBooks update → Reporting & delivery

This is the first workflow that should be implemented, tested, and kept stable as the foundation for all future enhancements.

---

## 1. High-Level Golden Path Overview

At a high level, the Golden Path looks like this:

1. Client is onboarded and connected (portal + bank + QuickBooks).
2. New transactions are retrieved from QuickBooks (or Plaid).
3. Transactions are sent to the AI Layer for categorization.
4. The AI suggests categories with confidence scores.
5. High-confidence results are posted automatically to QuickBooks.
6. All categorized data is logged to Google Sheets.
7. At month-end, a report is generated and delivered to the client.

No AP/AR automation, no forecasting, no complex edge cases.  
Just **clean, automated categorization + reporting** for one client.

---

## 2. Step-by-Step Golden Path

### 2.1 Step 1 – Client Onboarding

**Goal:** Register a new business as a client and configure system access.

**Actions:**

1. Create a client record in the **Client Portal** (SuiteDash or equivalent).
2. Connect their **QuickBooks Online company**:
   - OAuth flow to authorize access.
   - Store the QBO company ID and tokens securely.
3. Optionally connect **Plaid**:
   - Use Plaid Link to connect the client’s bank/credit accounts.
   - Store Plaid access token securely (not in GitHub).
4. Create or assign a **client-specific configuration** in `/config`:
   - Chart of accounts mapping
   - Custom category rules (e.g., specific vendor → specific category)

At the end of this step, the platform knows:
- Who the client is  
- Which QBO company to use  
- Where to fetch transactions from  
- What rules to apply  

---

### 2.2 Step 2 – Transaction Ingestion

**Goal:** Retrieve all relevant transactions that need categorization.

**Source of truth:** QuickBooks Online (possibly supplemented by Plaid).

**Actions:**

1. A **scheduled workflow** runs in Make.com (for example, daily).
2. The workflow calls QBO:
   - List bank/credit transactions for a date range (e.g., last 1–7 days).
   - Filter for **uncategorized** or **recently imported** transactions.
3. The raw transaction data is normalized into a consistent internal format, including:
   - Date  
   - Amount  
   - Description  
   - Vendor (if available)  
   - Account/origin (e.g., bank or card)  
   - Transaction ID  

The normalized batch is then passed forward to the AI Layer.

---

### 2.3 Step 3 – AI Categorization (GPT)

**Goal:** Have AI suggest the best accounting category for each transaction.

**Actions:**

1. The normalized batch is sent to the AI Layer via OpenAI’s API.
2. A **prompt template** in `/prompts/categorize_transactions.txt` is used, which includes:
   - A list of allowed categories (from the client’s chart of accounts).
   - Instructions to choose the single most appropriate category.
   - Instructions to return a **confidence score** (e.g., 0.0–1.0).
3. The AI returns, for each transaction:
   - Suggested category
   - Optional subcategory
   - Memo/description
   - Confidence score

Example (conceptual) response per transaction:
```json
{
  "transaction_id": "123",
  "suggested_category": "Meals and Entertainment",
  "memo": "Team lunch at restaurant",
  "confidence": 0.93
}
2.4 Step 4 – Confidence-Based Routing

Goal: Decide which transactions can be auto-posted and which need review.

Actions:

The Automation Layer (Make.com) evaluates each transaction:

If confidence >= 0.80 → mark as auto-approve

If confidence < 0.80 → mark as needs_review

For the Golden Path, we assume:

Most transactions are high-confidence.

Low-confidence transactions are relatively few and will be queued separately for manual review (outside of this “happy path”).

Both approved and review-flagged entries are logged to a Google Sheet:

transactions_log tab for the full history

review_queue tab for low-confidence items

For the Golden Path, the primary focus is the auto-approved flow.

2.5 Step 5 – Posting to QuickBooks

Goal: Write the categorized results back into QuickBooks automatically.

Actions:

For each auto-approved transaction:

Use QuickBooks Online API to update the transaction’s category/account.

Optionally update memo/description as well.

Confirm success or log any errors:

Successes are logged with a timestamp.

Failures are logged for retry or manual handling.

At the end of this step, the client’s QuickBooks company:

Has correctly categorized transactions

Matches the AI’s decisions for high-confidence items

2.6 Step 6 – Logging & Auditing

Goal: Maintain a full audit log of what the system did.

Actions:

All categorized transactions (approved + low-confidence) are logged to a Google Sheet, including:

Date

Description

Original category (if any)

Suggested category

Confidence

Status (auto-approved vs needs_review)

QBO sync status

This log is critical for:

Debugging issues

Explaining behavior to clients

Continuous improvement of AI prompts and rules

2.7 Step 7 – Month-End Reporting

Goal: Generate a clear, automated financial report at period-end.

Actions:

At month-end, a separate Make.com scenario runs:

Confirms that all transactions for the period are categorized.

Ensures there are no major anomalies (optional for Golden Path).

Financial data is pulled from QBO or the aggregated Google Sheet:

Totals by category

Key metrics (e.g., revenue, expenses, profit)

Data is written into a Google Sheets report tab or template file.

Optionally, Looker Studio dashboards are refreshed.

The Golden Path assumption:

Data is clean

No missing receipt edge cases

No conflicts or API failures

2.8 Step 8 – Client Delivery

Goal: Deliver the results to the client in a professional, automated way.

Actions:

The platform generates:

A PDF export of the report, or

A link to a live dashboard, or both.

The Notification layer (Mailchimp or equivalent) sends:

“Your books are ready” email

Includes:

Summary of the month

Link to portal

Attachments or dashboard URLs

The Client Portal is updated:

Reports and files are available for download.

Client can log in and review.

At this point, the full end-to-end Golden Path has executed successfully.

3. Golden Path vs. Edge Cases

The Golden Path does not include:

Handling API downtime (QuickBooks, Plaid, OpenAI, OCR)

Low-confidence transactions and manual review flows

Missing receipts and follow-up requests

AP/AR automation

Cash flow forecasting

Multi-entity consolidations

These are considered extensions to the core Golden Path and can be layered on once the primary flow is stable.

4. Developer Implementation Priority

When building or extending the system, developers should:

Protect the Golden Path first.
Never break the core ingestion → AI → QBO → reporting loop.

Add features as optional branches from this central flow.

Keep configuration (e.g., thresholds, mappings) externalized in /config so the Golden Path logic itself remains stable.

This Golden Path serves as the reference workflow for all design, testing, and iteration.
If a new feature or change conflicts with this flow, the impact should be carefully evaluated.
