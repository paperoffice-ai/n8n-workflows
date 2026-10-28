# Create Lexware Office vouchers from Gmail invoice attachments with PaperOffice AI

**Categories:** Document Ops, Finance, AI
**Tags:** invoice, OCR, IDP, Gmail, Lexware Office, lexoffice, accounts payable, bookkeeping
**Template code:** `dist/gmail-invoices-to-lexware-office.json`

---

Stop typing supplier invoices into Lexware Office. Every new Gmail message with a PDF or image attachment is read by PaperOffice AI document AI, and a voucher with the extracted data and the original file is created in Lexware Office with the status "to be checked" (Zu prüfen).

## Who is this for?
Small businesses, freelancers and bookkeepers in Germany who use Lexware Office (formerly lexoffice) and receive supplier invoices by email.

## What it does
- Watches Gmail for new emails with attachments (PDF, PNG, JPEG, TIFF) and skips small logo or signature images
- Sends each attachment to PaperOffice AI for structured invoice extraction (OCR plus AI, scans and photos included)
- Creates a purchase invoice voucher (or a purchase credit note for negative totals) in Lexware Office with supplier name, invoice number, invoice date, due date, gross and VAT amounts, and attaches the original file
- Leaves every voucher in the status "to be checked", so a person confirms category and booking in Lexware Office
- Skips invoices that already exist in Lexware Office with the same invoice number, and repeats of an invoice within the workflow
- Does not create vouchers for invoices that are not in EUR, have no invoice number or whose net and VAT amounts do not add up; those are listed as **Needs review** with a note. Invoices with other missing fields are sent and marked **Needs review**

## How it works
1. **Gmail Trigger** polls for new emails with attachments and downloads them.
2. **Split attachments** creates one item per invoice file.
3. **Extract invoice data** calls `POST https://api.paperoffice.ai/latest/job/add/workflow` with the extraction profile `invoice`. Running jobs are checked every 10 seconds.
4. **Build Lexware voucher** maps the fields and decides whether the invoice can be sent.
5. **Find invoice in Lexware** looks up the invoice number in the Lexware voucher list.
6. **Create voucher** and **Attach original file** create the voucher with the status "to be checked" and upload the file.
7. **Result** lists every invoice with its status and a direct link to the voucher in Lexware Office.

## Requirements
- A PaperOffice AI account and a User token (`po_ut_…`) from Account → API
- A Lexware Office plan with access to the Lexware Office Public API, and a private API key from https://app.lexware.de/addons/public-api
- A Gmail credential in n8n

## How to set up
1. Create a **Bearer Auth** credential with your PaperOffice AI token and select it in **Extract invoice data** and **Check job status**.
2. Create a second **Bearer Auth** credential with your Lexware Office API key and select it in **Find invoice in Lexware**, **Create voucher** and **Attach original file**.
3. Connect Gmail in **New email with attachment**. Narrow the search query, for example with `label:invoices`.
4. Optional: `CATEGORY_ID` in **Build Lexware voucher** is preset to the booking category "Sonstige Ausgaben". Change it if most of your invoices belong to another category; you confirm the category for each voucher in Lexware Office anyway.
5. Send yourself a test invoice and activate the workflow.

## How to customize
- Add a Slack or email message for invoices with the status **Needs review**, **Duplicate** or **Failed**.
- Use another trigger (Outlook, a folder, a form) in front of **Split attachments**.

## Limitations and manual attention
- Vouchers are always created as "to be checked". Category, contact and booking are confirmed by a person in Lexware Office; nothing is booked or paid automatically.
- The supplier is set as the name on the collective vendor contact. Vendor contacts in Lexware Office are not created or changed.
- Lexware Office vouchers are EUR only. Invoices in other currencies are listed as **Needs review** and not sent.
- Invoices without a readable invoice number cannot be checked against Lexware Office for duplicates and are listed as **Needs review** without a voucher.
- If Lexware Office does not answer in time when a voucher is created, the invoice is listed as **Uncertain** and not sent again automatically. Check Lexware Office before sending it again.
- Files larger than 5 MB are not accepted by Lexware Office.

No specific plan is required. Usage is billed to your PaperOffice AI account as usual.

Document processing (OCR, extraction, classification) runs on PaperOffice's own hardware in its own data center in Navarra, Spain (EU), without external AI providers. API reference for AI coding agents: https://api.paperoffice.ai/latest/docs/llms.txt
