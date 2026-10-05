# Test Results — QuickBooks AI Operations Assistant

> **Fictional / Sample Data.** All tests ran against an Intuit QuickBooks Online sandbox company, fictional expenses and Intuit's built-in sample customers and invoices. Execution numbers refer to the n8n execution log. Test emails went only to a test inbox, marked `[FICTIONAL TEST]`.

Testing period: October 2026. Every result below was checked in the n8n execution data and in Airtable, not assumed.

## 1. Transaction intake (Lane 1)

| Test | Result |
|---|---|
| First intake of 30 fictional expenses in the holding account | 30 queued for review, 9 flagged, 10 exception records created |
| Re-run with no new expenses | 0 duplicates created |
| QuickBooks connection failure (expired authorization simulated) | Step failed safely, alert email sent, error logged |
| Airtable failure (permission error simulated) | Step failed safely, alert email sent, error logged |

## 2. AI review and decision rules (Lane 1b)

| Test | Result |
|---|---|
| Jev reviewed 29 items | 20 to human review, 9 exceptions, 7 low-confidence exceptions created |
| Observations (fictional data, not an accuracy claim) | Clear vendors scored 92–98; vague ones 25–45. Jev raised fixed-asset concerns on a $1,250 desk and a $7,800 compressor. |
| AI failure | Item routed to human review with the reason recorded |

## 3. Approval sync to QuickBooks (Lane 2)

| Test | Result |
|---|---|
| Reviewer decisions (Approved, Changed, Rejected) | Approved and Changed updated in QuickBooks (category only). Rejected left untouched. |
| Auto-approval end to end (new fictional fuel expense) | Intake → Auto-Approved at confidence 98 → category updated in QuickBooks |
| Expense already categorized by someone else | Update refused ("no longer in the holding account"), logged, alert sent |
| Bad reviewer input: "Changed" with no category; "Approved" with a category filled (execution 2708) | Neither sent to QuickBooks. Both marked Sync Failed with a plain-language reason, audit records written, alert sent. |

## 4. Receipts and document requests (Lane 3)

| Test | Result |
|---|---|
| First receipt check (run 1382) | 17 under the $100 threshold (Not Required), 8 receipts found in Drive, 6 missing. 6 Jev drafts created at Pending Approval. Attention email sent. |
| Approval and send (execution 2637) | The 2 approved requests were emailed (the reviewer's edited text was used) and set to Requested with a follow-up date. Cancelled and unapproved requests were untouched. |
| Receipt arrives (execution 2640) | Request changed to Received with the document link. The expense changed to Receipt Status Found. |
| Follow-up date passed (execution 2640) | Request changed to Escalated, and the team was emailed |

## 5. Overdue invoice follow-up (Lane 5)

| Test | Result |
|---|---|
| First run (execution 2655) | 15 overdue invoices: 9 tracked, 5 within the 3-day grace period, 1 below $25. 8 reminder drafts (all written by Jev). The invoice 68 days overdue was escalated. |
| Follow-up due (execution 2658) | Second, firmer reminder drafted on the same record (no duplicate) |
| Two reminders already sent | Escalated to a person |
| Partial payment | Noted ("balance changed from $300.00 to $239.00") |
| On Hold | Left unchanged |
| Invoice no longer open in QuickBooks | Paid / Closed, reminders stopped |
| Approval and send (execution 2667) | 2 approved reminders emailed (reviewer's edits used). Reminder Sent, count 1, next check +7 days, audit written. |
| Email send failure (execution 2710) | After 3 tries: Send Failed with a note, reminder count unchanged, alert sent |

## 6. Weekly report (Lane 6)

| Test | Result |
|---|---|
| First report (execution 2673) | Figures matched Airtable. Issue found: deliberate failure tests and a safety-check refusal were all counted as "errors", and the AI guessed a cause. |
| After fix (execution 2675) | Errors, failed emails and safety-check refusals counted separately. The summary states counts only and points to the Audit Log. |

## 7. System-level failures

| Test | Result |
|---|---|
| Airtable failure alert after message cleanup (execution 2713) | Alert email and Audit Log record contain only the status and the Airtable message: no headers, cookies or tokens |
| Live crash (execution 2715, temporary failing step) | Crash alert ran (2716) and emailed the workflow, execution ID, failing step and error. The temporary step was removed afterwards. |

## Not separately tested

- Receipt-request **send failure** uses the same logic as the invoice-reminder send failure (tested above) but was not run on its own.

---

Michael Mendoza • mjmendoza.workph@gmail.com
