# QuoteFlow AI

AI-powered quotation request triage and human-in-the-loop response workflow for B2B service companies.

## Project Overview

QuoteFlow AI automates the early stages of the quotation request process for small and medium-sized B2B service companies.

Incoming customer emails are captured automatically, interpreted with AI, converted into structured data, evaluated using deterministic business rules, and displayed in an internal dashboard for employee review.

The system does not send customer-facing responses autonomously.

Employees can review, edit, approve, or reject the suggested reply before any external communication is sent.

---

## Business Problem

Many service companies receive quotation requests by email.

A typical employee must manually:

- read the customer email
- identify the requested service
- extract relevant requirements
- identify missing information
- decide whether the request is ready for review
- prepare a response
- send the response
- update the request status

This process is repetitive and time-consuming, especially when request volume increases.

---

## AS-IS Process

Customer email  
→ Employee reads message  
→ Manually identifies requirements  
→ Checks whether information is missing  
→ Prepares reply  
→ Sends email  
→ Tracks request manually

---

## TO-BE Process

Customer email  
→ n8n captures the message  
→ Raw request stored in Supabase  
→ Gemini extracts structured information  
→ n8n applies business rules  
→ Suggested reply is generated  
→ Data is displayed in Lovable dashboard  
→ Employee reviews or edits the reply  
→ Approve & Send  
→ n8n sends the email  
→ Supabase status updated to SENT

---

## How It Works

### 1. Email Intake

n8n monitors the company inbox using an Email Trigger.

The incoming email is normalized into:

- sender
- subject
- email body
- date

The raw request is immediately stored in Supabase with status:

NEW

This ensures that the request is stored before any AI processing occurs.

---

### 2. AI Requirement Extraction

Google Gemini analyzes the unstructured customer email and returns structured JSON.

Example:

{
  "company": "ABC Srl",
  "service": "Installation of a video surveillance system",
  "location": "Monza",
  "number_of_sites": 2,
  "deadline": "End of month",
  "budget": null,
  "site_size": null,
  "summary": "ABC Srl requests installation of a surveillance system for two warehouses in Monza.",
  "missing_information": [
    "budget",
    "site_size"
  ]
}

Gemini is instructed not to invent missing information.

---

## Business Rules

AI is used to interpret the customer request.

Workflow decisions are handled by deterministic n8n rules.

Example:

IF missing_information.length > 0  
→ NEEDS_INFO

ELSE  
→ READY_FOR_REVIEW

This separates AI interpretation from business logic.

---

## Suggested Reply

n8n generates a standard suggested reply based on the structured data.

If information is missing, the system asks the customer only for the missing information.

If the request is complete, the system confirms that the request is ready for internal review.

The suggested reply is stored in Supabase and displayed in the internal dashboard.

---

## Human-in-the-Loop

The employee retains control before any customer-facing communication is sent.

The dashboard allows the employee to:

- review the original customer email
- review the structured data
- read the AI summary
- see missing information
- edit the suggested reply
- reject the request
- approve and send the response

The final process is:

AI proposes  
→ Human reviews  
→ Automation executes

---

## Approval Workflow

When the employee clicks Approve & Send:

Lovable  
→ n8n Webhook  
→ Email address normalization  
→ SMTP Send Email  
→ Supabase Update  
→ status = SENT

The request is only marked as SENT after the email has been successfully transmitted.

---

## Error Handling

External AI services can temporarily become unavailable.

QuoteFlow includes basic failure handling.

The Gemini node automatically retries failed requests.

Configuration:

- Max retries: 3
- Wait between retries: 5 seconds

If Gemini still fails after the retries:

status = PROCESSING_FAILED

The customer request remains stored in Supabase and is not lost.

This allows failed requests to remain visible for manual review or later reprocessing.

---

## Request Statuses

The workflow uses the following statuses:

- NEW
- NEEDS_INFO
- READY_FOR_REVIEW
- PROCESSING_FAILED
- SENT
- REJECTED

---

## Architecture

Customer Email  
↓  
n8n Email Trigger  
↓  
Normalize Email  
↓  
Supabase Insert  
status = NEW  
↓  
Gemini AI  

Success path:  
Gemini AI  
→ Structured Data  
→ Business Rules  
→ Supabase  
→ Lovable UI  
→ Human Review  
→ Approve & Send  
→ n8n Webhook  
→ SMTP Email  
→ Supabase status = SENT

Error path:  
Gemini AI  
→ Retry x3  
→ PROCESSING_FAILED

---

## Tech Stack

- n8n — workflow orchestration
- Google Gemini — natural language interpretation
- Supabase — database and request state management
- Lovable — internal employee dashboard
- SMTP — outbound customer communication

No Python is required.

---

## Dashboard

The internal QuoteFlow dashboard includes:

### Overview

Request counts by status:

- New
- Needs Info
- Ready for Review
- Processing Failed
- Sent

### Requests

A list showing:

- Company
- Sender email
- Service
- Location
- Status
- Created date

### Request Detail

Employees can review:

- original email
- structured request data
- AI summary
- missing information
- suggested reply
- current status

Available actions:

- Approve & Send
- Save Edit
- Reject

---

## Business Value

QuoteFlow reduces manual work during the quotation intake process.

Potential benefits include:

- faster response times
- less manual email reading
- consistent request qualification
- fewer missed requirements
- standardized customer communication
- centralized request visibility
- employee control before sending responses

---

## Production Considerations

This project is an MVP and would require additional controls before use in a production environment.

Possible improvements include:

- authentication and user roles
- secure webhook authentication
- advanced retry and queue management
- AI provider fallback
- monitoring and alerting
- audit logs
- CRM integration
- company-specific qualification rules
- email templates
- manual reprocessing of failed requests
- rate-limit handling
- data retention policies

---

## Project Goal

The purpose of this project is not to build a complete CRM.

The goal is to demonstrate how AI, deterministic business rules, workflow orchestration, and human approval can be combined to redesign a real business process.

QuoteFlow demonstrates:

- AI extraction from unstructured communication
- workflow orchestration
- structured data management
- deterministic decision logic
- error handling
- human-in-the-loop approval
- system integration
- end-to-end business process automation
