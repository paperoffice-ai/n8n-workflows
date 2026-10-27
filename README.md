# n8n workflows for PaperOffice AI

Ready-to-import [n8n](https://n8n.io) workflows that use the [PaperOffice AI](https://paperoffice.ai/en/) API for document archiving, document AI and human review.

| Workflow | What it does |
|---|---|
| [Stop invoice fraud before payment](workflows/stop-invoice-fraud-before-payment.json) | Archives every incoming invoice, checks bank details, duplicates, order and delivery note, and sends it to a human reviewer. The workflow never releases a payment by itself. |

## Stop invoice fraud before payment

Catch changed bank details, duplicate invoices and invoices that do not match the order before anybody pays them.

**What it does**
- Archives the invoice PDF in your PaperOffice AI workspace and reads it with document AI (supplier, invoice number, amounts, IBAN, order number, line items).
- **IBAN check:** compares the IBAN with earlier invoices of the same supplier → `match`, `changed` or `unknown` (new supplier).
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
4. Activate the workflow. Copy the production URL of the **Review decision** node into the webhook URL of your HITL agent and save. Copy the signing secret that PaperOffice AI shows once into the **Compute signature** node (field *Secret*).
5. Upload your purchase orders and delivery notes to the same workspace, then submit a test invoice through the form.

All API calls cost credits in your PaperOffice AI account, including rejected calls. The **Settings** node checks your input before the first call.

**Customize**
- Replace the form with a Gmail, Outlook or Google Drive trigger.
- Send blocked invoices to Slack or Teams, or export approved ones to your ERP.
- Change the number of reviewers or add amount rules in the HITL agent.

## Security notes

- The workflow files contain **no credentials, tokens, webhook secrets or account data**. You add your own after the import.
- Keep the signing secret only in the **Compute signature** node or in an n8n credential. Do not share exported workflows that contain it.

## Links

- PaperOffice AI API reference (also for AI coding agents): https://api.paperoffice.ai/latest/docs/llms.txt
- Developer documentation: https://paperoffice.ai/en/developer/apis/

## License

[MIT](LICENSE)
