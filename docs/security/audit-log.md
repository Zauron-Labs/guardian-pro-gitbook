# Audit log

Each Guardian Pro site keeps its own audit log, the audit trail for HIPAA 45 CFR 164.312(b).

## Integrity and retention

- **Append-only, enforced by the database.** Updates, deletes and truncation of the audit log are refused by the database itself, not only by the application.
- **Kept for at least six years and never purged.** No retention setting applies to it (see [Data retention](data-retention.md)).
- **Patient data is never released unaudited.** For study views, single-case patient-data reads, patient data through the Guardian API and the audit log export, the audit entry is written first. If it cannot be written, the request fails and nothing is returned.
- **Write failures are visible.** If any other audit entry cannot be written, Guardian logs the error and raises a System alert for your site, so a gap in the log does not go unnoticed.

## What is recorded

- Sign-ins, failed sign-ins and sign-outs
- Study views in the viewer (one entry per link and study every 30 minutes)
- Single-case patient-data reads, such as opening a Flag Case or a prior report
- Exports, including the audit log export itself
- Patient-data and identifier reads through the [Guardian API](../guardian-api.md), and API key creation and revocation
- Task launches from the task feed
- **Every administrative change.** Every change made through the Admin tab or the admin settings: users added or imported, roles, access expiry, sign-in and directory settings, integrations, notification and review settings, deletions. A change that has its own specific entry (for example a role change or an export) is recorded once, by that entry.
- **Every refused request.** Every request to Guardian's APIs refused for a missing sign-in or a missing permission, with the outcome **denied**. To keep a scanner from flooding the log, repeats of the same refusal (same person, address and request) are recorded once a minute, and the next entry says how many were left out.
- Refusals from outside your [admin allowed networks](../security.md#access-control)
- Zauron support sessions: when they start and end, and every read and change made during them

**Never recorded:** request or response bodies, query strings, passwords, tokens (a session appears only as a short one-way hash), report text, patient names or medical record numbers.

## Record format

Every entry has the same fields in the export and the stream (format name `guardian.audit.v1`):

| Field | Meaning |
|---|---|
| `schema` | Always `guardian.audit.v1` |
| `table` | Which log the entry comes from: your site's log, or the platform log for Zauron operator actions |
| `id` | Entry number in that log, increasing |
| `tenant` | Your site's identifier |
| `created_at` | Time of the entry, ISO-8601 in UTC |
| `actor` | Who acted: a Guardian user, an API key, a Zauron operator, or anonymous |
| `role` | Their role at the time |
| `action` | What happened, for example `sign_in`, `phi_read`, `phi_export`, `admin_put`, `api_access_denied` |
| `outcome` | `success`, `failure` or `denied` |
| `resource_type` | What was acted on, for example a study, an export, an admin setting or a review |
| `resource_id` | Which one: the accession, Study Instance UID, review number, or the request route (never the raw address with its parameters) |
| `session_id` | One-way hash of the session, never the session token |
| `source_ip` | The client's address |
| `detail` | Route, status, record numbers and counts; never request bodies or patient text |
| `stored` | Stream only: `false` when the entry could not be saved in the database |

Entries written before the outcome field existed show the outcome their action implies (for example, an action ending in `_failed` reads as `failure`).

## Export

Administrators can download the log for any period as JSON lines or CSV, oldest entry first:

```
GET https://<your Guardian address>/api/admin/audit/export?since=2026-11-01&until=2026-11-18&format=ndjson
```

- Needs a signed-in administrator session.
- `since` and `until` are ISO-8601 dates or date-times in UTC. A date alone in `until` includes that whole day. Without them, the last 7 days are exported.
- `format` is `ndjson` (default) or `csv`.
- `after_id` continues after the last entry a previous export returned, so a scheduled pull can ship the log to your SIEM incrementally.
- Up to 1,000,000 entries per request. The `X-Audit-Rows` response header gives the count.
- The export is itself recorded as an export. If that record cannot be written, the export does not start.

## Streaming to your SIEM (dedicated installations)

A dedicated installation can also copy every audit entry, as it is written, to your log pipeline as one JSON line per entry:

- **Standard output** of the Guardian container, for a Docker log driver such as `awslogs` (Amazon CloudWatch Logs), `syslog` or `fluentd`. Filter on `schema = "guardian.audit.v1"` to pick the audit lines out of the container log.
- **A file** on the Guardian host, for the CloudWatch agent, Fluent Bit or rsyslog to collect. The file works with log rotation.

Things to know:

- The stream is a copy; the database log stays the record. Entries that could not be saved in the database are still streamed, marked `stored: false`.
- In the stream, the part of an email address before the `@` is masked. The export shows it in full.
- Study views from the viewer and DataForge's patient-data reads and exports are saved directly to the database and are not in the stream. For a complete feed, use the export with `after_id`, alone or alongside the stream.

Ask Zauron to turn the stream on, and tell us whether you want standard output or a file.

## Questions

Contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
