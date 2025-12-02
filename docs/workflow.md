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
