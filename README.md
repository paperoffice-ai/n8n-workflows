# n8n workflows for PaperOffice AI

Ready-to-import [n8n](https://n8n.io) workflows that use the [PaperOffice AI](https://paperoffice.ai/en/) API for document archiving, document AI and human review.

| Workflow | What it does |
|---|---|
| [Stop invoice fraud before payment](workflows/stop-invoice-fraud-before-payment.json) | Archives every incoming invoice, checks bank details, duplicates, order and delivery note, and sends it to a human reviewer. The workflow never releases a payment by itself. |
| [AP inbox: batch scans to approved invoices](workflows/ap-inbox-split-three-way-match-human-approval.json) | Splits a scanned stack of orders, delivery notes and invoices, matches every invoice against the real order and delivery note, flags fraud signals and lets a human decide. Approved invoices are archived, rejected ones go to quarantine with a legal hold. |

## Stop invoice fraud before payment

Catch changed bank details, duplicate invoices and invoices that do not match the order before anybody pays them.

**What it does**
- Archives the invoice PDF in your PaperOffice AI workspace and reads it with document AI (supplier, invoice number, amounts, IBAN, order number, line items).
- **IBAN check:** compares the IBAN with earlier invoices of the same supplier → `match`, `changed` or `unknown` (new supplier). Invoices tagged `fraud-check:blocked` never count as known bank details; an IBAN that appears on any blocked invoice, under any supplier name, is a finding of its own (`blocked`).
- **Duplicate check:** PaperOffice AI duplicate detection plus a search for the same invoice number.
- **Order and delivery note:** finds the purchase order and the delivery note by the order number and compares quantities and totals.
- Writes a check report to the document and queues every invoice for human review (HITL). Findings are listed first.
- Receives the reviewers' decision by a **signed webhook** and tags the document `fraud-check:ready-for-payment` or `fraud-check:blocked`. Unsigned calls are rejected.
- Optional deep check with AI-DMS Ultra (paid, billed per page).

**Requirements**
- A self-hosted or cloud n8n instance (built and tested with n8n 2.x).
- A PaperOffice AI account with Workflow AI (HITL agents are part of Workflow AI; this is a paid feature) and your own API key: a User token (`po_ut_…`) from **Account → API** in the PaperOffice AI app.
- A HITL agent in PaperOffice AI (**Workflow AI → HITL agents**) bound to your invoice workspace.

**Set up (about 10 minutes)**
1. In n8n, choose **Import from file** and select `workflows/stop-invoice-fraud-before-payment.json`.
2. Create a **Bearer Auth** credential with your PaperOffice AI token and select it in every PaperOffice AI node.
3. Open the **Settings** node and enter your workspace ID and the UUID of your HITL agent.
4. Open the **Review decision** node and copy its production URL into the webhook URL of your HITL agent and save. Copy the signing secret that PaperOffice AI shows once into the **Compute signature** node (field *Secret*).
5. Activate the workflow. n8n activates it only once the secret is set.
6. Upload your purchase orders, delivery notes and a few earlier paid invoices to the same workspace, then submit a test invoice through the form.

All API calls cost credits in your PaperOffice AI account, including rejected calls. The **Settings** node checks your input before the first call.

**Customize**
- Replace the form with a Gmail, Outlook or Google Drive trigger.
- Send blocked invoices to Slack or Teams, or export approved ones to your ERP.
- Change the number of reviewers or add amount rules in the HITL agent.

**Limitations and manual attention**
- The workflow stops with a clear message if the upload is rejected (for example an invalid token). Nothing is queued in that case; submit the invoice again.
- An invoice whose fields could not be read is still queued, with the finding "could not be read completely".
- Documents uploaded seconds before are found by full-text search only after indexing.
- Without earlier invoices of a supplier in the workspace, every IBAN is "unknown" and goes to review as a finding.
- An invoice is tagged only after the reviewers decide. Until then it is neither "ready for payment" nor "blocked".


## AP inbox: batch scans to approved invoices

Turn a scanned stack of purchase orders, delivery notes and invoices into checked, reviewer-approved and archived documents. The workflow never releases a payment by itself.

Video walkthrough: https://www.youtube.com/watch?v=gDwfCGvXyjo

**What it does**
- **Splits batch scans:** one PDF with several documents is separated into single documents (split model `premium` returns document type, date and sender).
- Archives and reads every document in your inbox workspace: invoices, purchase orders and delivery notes with line items.
- **Three-way match against real documents:** the order and delivery note come from the same scan or from your archive. Quantities are compared per item code.
- **Fraud checks:** IBAN against approved earlier invoices (an IBAN that appears on any rejected invoice, under any supplier name, is a finding of its own), duplicate invoice numbers, the same order already billed, and all invoices above the order total.
- Stores a check report on the invoice and queues it for human review (HITL). The decision comes back by a **signed webhook**.
- **Archive by decision:** approved invoices move to your approved workspace (optional WORM lock), rejected ones move to quarantine with a legal hold.

**Requirements**
- A self-hosted or cloud n8n instance (built and tested with n8n 2.x).
- A PaperOffice AI account with Workflow AI and a User token (`po_ut_…`) from **Account → API**. All PaperOffice AI calls are paid.
- Three workspaces (inbox, approved, quarantine) with AI-DMS set to Basic or higher (paid per document, in addition to the extraction this workflow runs).
- A HITL agent bound to the inbox workspace. Keep its review fields optional.
- Earlier paid invoices in the approved workspace: only they count as known bank details.

**Set up**
1. In n8n, choose **Import from file** and select `workflows/ap-inbox-split-three-way-match-human-approval.json`.
2. Create a **Bearer Auth** credential with your PaperOffice AI token and select it in every PaperOffice AI node.
3. In **Settings**, enter the inbox, approved and quarantine workspace IDs and the HITL agent UUID. In **Archive settings**, enter the approved and quarantine IDs again.
4. Open the **Review decision** node and copy its production URL into the webhook URL of your HITL agent and save. Copy the signing secret that PaperOffice AI shows once into the **Compute signature** node (field *Secret*).
5. Activate the workflow (n8n activates it only once the secret is set), then upload a scan with an order, a delivery note and an invoice.

## Security notes

- The workflow files contain **no credentials, tokens, webhook secrets or account data**. You add your own after the import.
- Keep the signing secret only in the **Compute signature** node or in an n8n credential. Do not share exported workflows that contain it.

## Links

- PaperOffice AI API reference (also for AI coding agents): https://api.paperoffice.ai/latest/docs/llms.txt
- Developer documentation: https://paperoffice.ai/en/developer/apis/

## License

[MIT](LICENSE)
