# Data retention

How long Guardian Pro keeps each kind of data. Your PACS and reporting system stay the systems of record: Guardian keeps working copies, and the settings below decide when they are removed.

Retention is set per customer. Zauron sets the values for your site at your request and can send you the current values at any time. A cleanup job applies them continuously.

## Database records

These three settings are **off by default** (records are kept indefinitely), so nothing is deleted until your organization chooses a period. Any period you set is at least 30 days.

| Data | What is removed | Never removed while |
|---|---|---|
| **Report text** | The report text of an exam, its search index entries, the text quoted as evidence for quality measures, and Guardian's copy of the HL7 message, a chosen number of days after the report was processed. The exam itself, its codes, AI results and metrics stay. | A review, review link, champion alert, open task, comparison, export or DataForge study still needs the case; and never inside your review window or the period Guardian looks back when choosing cases |
| **Review records** | Peer reviews and Flag Cases, with their AI feedback, a chosen number of days after their last activity. Compliance counts no longer include them afterwards. | The review is still open (unresolved champion alert, unsent notification, open task); and never inside your review window or the case-selection look-back |
| **Activity and email log** | Entries in **Admin → Logs** (sent emails with their content, Flag Case and viewer opens, rejected HL7 messages, task feed events), a chosen number of days after they were written | — |

If Guardian cannot check whether cases are still needed, it removes no report text or review records in that pass and tries again on the next one.

## Patient identifiers in HL7 messages

Guardian keeps the original HL7 message (with patient name, medical record number, date of birth and address) only until the report is handled. Once the report is ingested, replaced or cancelled, the original message and the stored medical record number are erased after **30 days** by default (adjustable from 1 to 365 days). Messages still waiting, such as a preliminary report without its final, are kept for 180 days after receipt, or your period if it is longer. Matching a patient's exams does not depend on them.

## Images

Guardian's viewer copies of studies are removed on a schedule set for your site, and studies in use are kept longer. See [How long does Guardian keep study images](../faq.md#images-and-retention).

## Fixed periods

| Data | Retention |
|---|---|
| Viewer links in emails and shared from the dashboard | Stop working after 28 days |
| Peer Learning PowerPoint links | Up to 28 days |
| Audit log | At least six years; never purged (see [Audit log](audit-log.md)) |
| Other case records (AI results, codes, structured findings, ratings, notification receipts) | Kept until your site's data is removed at the end of your agreement |

## End of agreement

When your agreement ends, your data is returned or deleted as the agreement describes. On Guardian Pro Cloud, your site's database is removed after the agreed export hand-off.

## Questions

To change a retention period, contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
