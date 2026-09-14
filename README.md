# intelligent-banking-document-automation_capstone
AI-powered banking document automation system using OCR, LLMs, FastAPI, and RAG to classify documents, extract customer information, validate mandatory fields, detect missing documents, and generate structured JSON responses.
Banking OCR Automation

AI-powered banking document automation workflow that uses OCR, OpenAI LLMs, n8n workflow automation, validation, email notification, and PostgreSQL to process banking forms and convert unstructured documents into structured JSON.

The workflow receives a banking form through a webhook, sends the document to an OCR service, classifies the form using an OpenAI model, routes it to the appropriate extraction agent, validates mandatory fields, generates an acknowledgement ID, and stores the processed request in PostgreSQL.

**Key Features**
Banking document upload
OCR-based text extraction
AI-based document classification
Form-specific information extraction
Mandatory-field validation
Missing-field detection
Complete/incomplete application handling
Acknowledgement ID generation
Email notification
PostgreSQL persistence
Structured JSON extraction
n8n workflow orchestration
Supported Banking Forms

The current AI classifier supports the following banking forms:

Account Opening Form
ATM/Debit Card Block or Replacement Request
Cheque Book Request Form
Address Change Request Form
RTGS/NEFT Fund Transfer Form
KYC Update Form
Locker Access / Surrender Request Form
Architecture
Customer / Banking User
          |
          v
     n8n Webhook
          |
          v
      OCR Service
          |
          v
    Document Text
          |
          v
  AI Form Classifier
          |
          v
     Form Routing
          |
    +-----+-----+-----+
    |     |     |     |
    v     v     v     v
 Account ATM   KYC  RTGS/NEFT
 Opening Card Update Transfer
    |
    +---- Other Form Agents
          |
          v
   Structured JSON
          |
          v
      Validation
          |
       +--+--+
       |     |
       v     v
    Missing Complete
       |     |
       v     v
     Email  ACK ID
       |     |
       +--+--+
          |
          v
      PostgreSQL
          |
          v
     Final Response

The documented flow describes the stages as upload, local file handling, Docling OCR, OpenAI/Gemini classification, form routing, field extraction, validation, missing/complete branching, email/acknowledgement processing, database update, and final processing.

Technology Stack
Component	Technology
Workflow Automation	n8n
Programming	Python
OCR	Docling
LLM	OpenAI GPT-5-mini
AI Agents	n8n LangChain AI Agents
Database	PostgreSQL
Database Format	JSONB
Email	Gmail
API	Webhook / HTTP
Output	Structured JSON

The n8n workflow uses an upload webhook and calls an OCR endpoint at /extract; the configured OpenAI chat model is gpt-5-mini.

Document Upload

The workflow starts with an HTTP POST webhook using the path:

/upload-form

The uploaded file is then passed to the OCR service through an HTTP multipart request.

AI Document Classification

After OCR, the extracted document text is passed to an AI classifier.

The classifier determines:

{
  "form_type": "",
  "confidence": "HIGH|MEDIUM|LOW",
  "reason": ""
}

The result is then used by the routing logic to select the appropriate form-specific extraction agent.

Form-Specific AI Extraction

Each supported form has its own extraction prompt and JSON schema.

Account Opening

Extracts information such as:

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

Mandatory fields are checked and the application is marked COMPLETE or INCOMPLETE.

RTGS/NEFT

Extracts:

Transfer type
Customer name
Debit account number
Beneficiary name
Beneficiary account number
Beneficiary bank
IFSC
Transfer amount
Currency
Payment purpose
Transaction date
Mobile number
Charges
Signature

The supported transfer types are RTGS and NEFT.

KYC Update

Extracts:

Customer information
Account number
PAN
Aadhaar
Mobile number
Email
Address
Occupation
Income
KYC update type
Documents submitted
Signature

The workflow also supports multiple KYC update types and multiple submitted documents.

ATM/Debit Card

The extraction workflow handles:

Customer name
Account number
Last four card digits
Request type
Reason
Registered mobile number
Email
Branch
Request date
Signature
Cheque Book

The workflow extracts:

Customer name
Account number
Account type
Branch
Number of leaves
Delivery mode
Mobile number
Email
Request date
Signature
Validation

After AI extraction, the workflow checks the missing_fields array.

missing_fields.length > 0

If mandatory information is missing:

INCOMPLETE
     |
     v
Generate Acknowledgement ID
     |
     v
Send Email

If all required information is available:

COMPLETE
     |
     v
Generate Acknowledgement ID
     |
     v
Store in PostgreSQL
     |
     v
Send Acknowledgement Email

The workflow explicitly branches based on whether missing fields exist.

Acknowledgement ID

The workflow generates acknowledgement IDs in the format:

ACK-HDFC-YYYYMMDD-XXXX

The application status is set based on the validation result.

Email Notification

The workflow sends acknowledgement emails for both missing-field and complete applications.

For incomplete requests, the email includes the pending/missing information and acknowledgement ID.

For completed requests, the acknowledgement ID and form type are included in the email.

PostgreSQL Database

Processed banking form requests are stored in:

banking_form_requests

The table contains:

Column	Type
id	integer
acknowledgement_id	varchar(50)
form_type	varchar(100)
application_status	varchar(50)
extracted_json	jsonb
missing_fields	jsonb
created_at	timestamp

The acknowledgement ID is unique and the id column is the primary key.

Data Storage

The workflow stores the complete extracted JSON in the extracted_json JSONB column and the missing fields separately as JSONB. The SQL agent is configured to execute the generated PostgreSQL statement through the Postgres tool.

Example Extracted JSON
{
  "form_type": "RTGS/NEFT Fund Transfer Form",
  "transfer_type": "NEFT",
  "customer_name": "Rajesh Kumar",
  "debit_account_number": "50101122334455",
  "beneficiary_name": "Anita Sharma",
  "beneficiary_account_number": "98765432100123",
  "beneficiary_bank_name": "State Bank of India",
  "beneficiary_ifsc_code": "SBIN0001234",
  "transfer_amount": "250000",
  "currency": "INR",
  "missing_fields": [],
  "validation_status": "COMPLETE"
}
End-to-End Processing
1. Upload Banking Form
          ↓
2. n8n Webhook
          ↓
3. OCR Processing
          ↓
4. Extract Document Text
          ↓
5. AI Form Classification
          ↓
6. Form Type Routing
          ↓
7. Form-Specific AI Extraction
          ↓
8. Structured JSON
          ↓
9. Mandatory Field Validation
          ↓
10. Missing / Complete Decision
          ↓
11. Generate Acknowledgement ID
          ↓
12. Email Notification
          ↓
13. PostgreSQL Storage
          ↓
14. Ready for Processing
