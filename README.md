# intelligent-banking-document-automation_capstone
An AI-powered banking document automation system that converts uploaded banking forms into structured data using OCR, LLM-based document classification, intelligent field extraction, validation, email notifications, and PostgreSQL.

The workflow is orchestrated using n8n and uses OpenAI-based AI agents for document understanding.

🚀 **Project Overview**

Traditional banking form processing requires manual verification and data entry.

This solution automates the process:

Upload → OCR → AI Classification → Data Extraction → Validation → Acknowledgement → Database

The system identifies the banking form, extracts the required fields, checks mandatory information, generates an acknowledgement ID, sends an email notification, and stores the processed request in PostgreSQL.

## 🏗️ Architecture

<img width="712" height="646" alt="image" src="https://github.com/user-attachments/assets/22aac187-3b17-4791-995e-2869c267f57d" />

      **  AI Capabilities**
1. **Document Classification**

The AI classifier analyzes OCR text and identifies the banking form type with a confidence level:

{
  "form_type": "Account Opening Form",
  "confidence": "HIGH",
  "reason": "Detected account opening related fields"
}

The current classifier supports seven banking form types.

2. **Intelligent Information Extraction**

After classification, the document is routed to a form-specific AI extraction agent.

For example, an Account Opening Form can extract:

Customer name
Date of birth
Account type
PAN
Aadhaar
Address
Mobile number
Email
Occupation
Annual income
Nominee details
Initial deposit
Signature

Mandatory fields are automatically validated.

	
Processing Flow
Step 1 — Upload

Customer uploads a banking document through the n8n webhook.

Step 2 — OCR

The uploaded document is sent to the OCR service and converted into text.

Step 3 — AI Classification

The extracted text is analyzed by the OpenAI model to determine the document type. The workflow is configured with gpt-5-mini.

Step 4 — Intelligent Routing

Based on the detected form type, n8n routes the request to the corresponding AI extraction agent.

Step 5 — Data Extraction

The selected AI agent extracts the relevant fields into structured JSON.

Step 6 — Validation

Mandatory fields are checked.

All mandatory fields present
          │
          ▼
       COMPLETE

or

Mandatory field missing
          │
          ▼
      INCOMPLETE
Step 7 — Acknowledgement

An acknowledgement ID is generated for the request.

Step 8 — Notification

The customer receives an email containing the acknowledgement information. The workflow has separate handling for missing-field and completed requests.

Step 9 — Database Storage

The processed request is stored in PostgreSQL.

**PostgreSQL**

The main table is:

banking_form_requests

| Column             | Type         |
| ------------------ | ------------ |
| id                 | integer      |
| acknowledgement_id | varchar(50)  |
| form_type          | varchar(100) |
| application_status | varchar(50)  |
| extracted_json     | jsonb        |
| missing_fields     | jsonb        |
| created_at         | timestamp    |

**Technology Stack**
Workflow Automation     → n8n
Programming             → Python
OCR                     → Docling
LLM                     → OpenAI GPT-5-mini
AI Agents               → n8n LangChain
Database                → PostgreSQL
Database Format         → JSONB
Email                   → Gmail
API                     → HTTP Webhook

**Banking_OCR_Automation.json**

**Complete n8n workflow containing:**

Webhook,
OCR integration,
AI classification,
Form routing,
AI extraction agents,
Validation,
Acknowledgement generation,
Email notification,
PostgreSQL integration,

**DDL.txt**

PostgreSQL table definition for storing processed banking requests.


🎯** Business Benefits :**
Reduces manual data entry,
Automates document classification,
Extracts structured information from unstructured forms,
Detects missing mandatory information,
Provides acknowledgement tracking,
Reduces processing time,
Standardizes banking document processing,
Creates a structured database record for downstream processing.
