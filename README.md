BookKeepXperts Platform
AI-Powered Automated Bookkeeping Engine for Small & Medium Businesses

BookKeepXperts is a modern automation-first bookkeeping system that replaces manual bookkeeping with a fully integrated workflow using:

AI (OpenAI GPT) for categorization, anomaly detection & summaries

OCR (Veryfi / Document AI) for receipt & invoice extraction

Automation (Make.com) to orchestrate all processes

Accounting API (QuickBooks Online + Plaid) for financial data sync

Google Sheets + Looker Studio for real-time dashboards

This repository contains the documentation, prompts, scripts, templates, and blueprints needed for developers to build, maintain, and extend the BookKeepXperts automation platform.

📚 Table of Contents

Overview

Architecture

Core Workflow

Repository Structure

Developer Setup

Environment Variables

AI Prompt Templates

Make.com Blueprints

Testing & Scripts

Security

Contributing

License

🚀 Overview

BookKeepXperts automates the entire bookkeeping process from ingestion → categorization → reconciliation → reporting.
The goal is to deliver:

Accurate, real-time financials

90–95% automation

High-margin recurring revenue

Scalable operations without hiring bookkeepers

This repo functions as the developer hub for the platform, containing:

source documentation

automation blueprints

configuration templates

AI prompt logic

testing scripts

workflow definitions

🧱 Architecture
flowchart TD
    A[Client Portal (SuiteDash)] --> B[Automation Layer (Make.com)]
    B --> C[AI Layer (OpenAI GPT)]
    B --> D[OCR Engine (Veryfi/DocAI)]
    C --> E[Categorization Logic Engine]
    D --> E
    E --> F[QuickBooks Online API]
    F --> G[Reporting Layer - Google Sheets]
    G --> H[Looker Studio Dashboards]

Architecture Layers
Layer	Description
Client Portal	User access, file uploads, communication
Automation Layer	Make.com orchestrates ingestion → AI → QBO sync
AI Layer	Categorization, anomaly detection, monthly summaries
OCR Layer	Receipt & invoice extraction
Accounting Layer	QuickBooks + Plaid API integration
Reporting Layer	Sheets → Looker dashboards
Notification Layer	Email/SMS delivery, status updates

See /docs/architecture.md for more details.

🔁 Core Workflow

Client Onboarding

Bank Feed Sync (QBO / Plaid)

OCR Processing of Receipts

AI Categorization via GPT

Confidence Scoring & Review Queue

Post Back to QuickBooks Online

Monthly Reporting

Dashboard Refresh (Sheets → Looker Studio)

Delivery to Client Portal

See /docs/workflow.md for deep documentation.

📂 Repository Structure

Your repo is correctly structured and matches best practices:

BookKeepXperts-platform/
│
├── README.md                  → main project documentation
├── .gitignore                 → excludes secrets & system files
│
├── docs/                      → architecture & workflow documentation
│   ├── architecture.md
│   ├── workflow.md
│   ├── onboarding.md
│   ├── security.md
│   ├── prompts.md
│   └── faq.md
│
├── prompts/                   → AI prompt templates
│   ├── categorize_transactions.txt
│   ├── monthly_summary.txt
│   ├── anomaly_detection.txt
│   └── vendor_rules_example.json
│
├── scripts/                   → API test scripts
│   ├── test_qbo_connection.py
│   ├── test_plaid_connection.py
│   ├── test_openai_prompt.py
│   ├── categorize_sample_batch.py
│   └── post_to_qbo_example.py
│
├── make-blueprints/           → Make.com blueprint exports
│   ├── categorization_scenario.json
│   ├── receipt_ocr_pipeline.json
│   ├── reporting_pipeline.json
│   └── ap_automation_pipeline.json
│
├── config/                    → configuration templates
│   ├── sample_chart_of_accounts.json
│   ├── mapping_rules_example.json
│   └── client_config_template.csv
│
├── dashboards/                → dashboard templates (PDF + CSV)
│   ├── looker_template_1.pdf
│   └── sheets_template.csv
│
└── tests/                     → unit & integration tests
    ├── unit_test_samples.py
    └── integration_test_samples.py

🛠 Developer Setup
Install dependencies

You will need:

Python 3.10+

Make.com account

QuickBooks Online Developer account

Plaid Sandbox account

Veryfi or Document AI account

OpenAI API key

Google Workspace

Clone the repository
git clone https://github.com/BookkeepX/BookKeepXperts-platform.git
cd BookKeepXperts-platform

Install Python dependencies (if needed)
pip install -r requirements.txt

Test API connections

In scripts/:

test_qbo_connection.py

test_plaid_connection.py

test_openai_prompt.py

Run one:

python scripts/test_qbo_connection.py

🔐 Environment Variables

You should create a .env file (not committed):

OPENAI_API_KEY=
QUICKBOOKS_CLIENT_ID=
QUICKBOOKS_CLIENT_SECRET=
QUICKBOOKS_REFRESH_TOKEN=
PLAID_CLIENT_ID=
PLAID_SECRET=
VERYFI_API_KEY=

🧠 AI Prompt Templates

Stored in /prompts/:

Categorization

Monthly summary

Vendor rules

Anomaly detection

Modify these with version control to maintain consistent behavior.

🔧 Make.com Blueprints

Blueprint JSON files are stored in /make-blueprints/.

Import them into Make.com → Scenarios → Import Blueprint.

Examples included:

Categorization engine

OCR processing

Reporting pipeline

AP automation

🧪 Testing & Scripts

The scripts/ folder includes tools to test:

QuickBooks auth

Plaid auth

OCR parsing

OpenAI categorization

Sample transaction batch processing

🛡 Security

🔑 Use .env for secrets

🚫 Never commit API keys

🔒 Enable GitHub secret scanning

🧩 Rotate keys every 90 days

🔐 Restrict developer access

Full security policy available in /docs/security.md.

🤝 Contributing

Fork the repo

Create a new branch

Make changes

Create a Pull Request

All PRs must pass test scripts

📜 License

Proprietary — All rights reserved.
Unauthorized copying or distribution is prohibited.
