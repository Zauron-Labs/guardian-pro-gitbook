# Guardian API

The Guardian API is for integrating your applications with Guardian Pro:

- **REST API (headless Guardian).** Your viewer, worklist or reporting system reads exams and AI findings and completes peer reviews through Guardian's `/v1` API, without Guardian's screens.
- **PACS Flag Case button.** A browser link that a button in your PACS or reporting software opens, so a radiologist can flag a case for peer review.

To integrate an AI model, so that it runs inside Guardian, see the [Model API](model-api.md) instead.

---

## REST API (headless Guardian)

### Turning it on

The REST API is **off by default** for every site.

1. Ask Zauron to switch on the Guardian API for your site. Until it is on, every `/v1` request is refused, whatever key is sent.
2. A Guardian administrator creates API keys under **Admin → Settings → Guardian API keys**.
3. Your integration calls `https://<your Guardian address>/v1/…` with a key.

Zauron can switch the API off again at any time. Keys stay listed, but no request is accepted until it is back on.

### API keys

| Setting | Rule |
|---------|------|
| Scopes | Each key carries only the scopes you select (table below). |
| IP allow-list | Required. Between 1 and 20 IP addresses or CIDR ranges that the key may be used from. A range that allows every address is refused. |
| Expiry | 30, 90, 180 or 365 days. |
| Active keys | At most 20 per site. |
| Secret | Shown **once**, when the key is created. Guardian stores only a hash, so a lost key can't be recovered: revoke it and create a new one. |
| Revoke | Takes effect on the next request. |

The REST API doesn't use your web access ranges; each key's IP allow-list applies instead. Creating and revoking keys is recorded in your audit log. Treat a key like a password: keep it in your integration's secret store and never put it in a URL, a browser page or source code.

**Scopes**

| Scope | Allows |
|-------|--------|
| `exams:read` | List exams and read their processing status |
| `findings:read` | Read AI findings (image detections and report findings) and the models in use |
| `phi:read` | Include patient-identifying details and report text in responses (see [Patient data](#patient-data)) |
| `reviews:write` | List, claim, score and defer peer review tasks |
| `reviews:finalize` | Champion final decisions on review tasks |

### Authentication

Send the key in the `Authorization` header on every request:

```
Authorization: Bearer <your API key>
```

- A key in the URL (query string) is refused, and cookies are ignored: a signed-in browser can't use the API.
- Requests from an address outside the key's IP allow-list are refused, even with a valid key.
- Each key is rate-limited. Repeated failed attempts from one address are blocked for a while.

The current OpenAPI description is at `GET /v1/openapi.json` (requires a key). `GET /v1/whoami` returns the calling key's name, scopes and expiry, never the secret.

### Reading exams and findings

| Endpoint | Scope | Returns |
|----------|-------|---------|
| `GET /v1/exams` | `exams:read` | Exams in the order Guardian received them, a page at a time: pass `next_cursor` from one page as `cursor` to get the next (it is `null` on the last page). Filters: `updated_since` (ISO 8601: only exams with a change at or after that time) and `limit` (1–200, default 50). |
| `GET /v1/exams/{accession}` | `exams:read` | One exam: accessions, procedures, image status, report status and the progress of report and image analysis. |
| `GET /v1/exams/{accession}/findings` | `findings:read` | AI image findings: ICD-10 code, confidence, series and image, a bounding box in normalized coordinates, the model, and whether the finding agrees with the report (`agrees`, `pending_review`, `confirmed` or `dismissed`). |
| `GET /v1/exams/{accession}/report-findings` | `findings:read` | Diagnoses found in the report (ICD-10, present / absent), critical-finding status and communication, and the teaching-case score. |
| `GET /v1/models` | `findings:read` | The AI models in use at your site: name, version, target condition, modalities, procedure codes and body parts. |

These are the same AI findings radiologists see in Guardian's viewer, with study, series and image identifiers so your viewer can show them on the right images.

### Peer review

Your viewer or worklist can complete Guardian peer reviews. Guardian still chooses the cases and assigns them to reviewers; your system shows the case and sends back the reviewer's decision. The same rules apply as in Guardian's own viewer.

Name the reviewer on every review request:

```
X-Guardian-Reviewer: <the reviewer's Guardian work email or NPI>
```

It must match an active Guardian user exactly; otherwise the request is refused (422). A reviewer only sees and acts on their own tasks.

| Endpoint | Scope | What it does |
|----------|-------|--------------|
| `GET /v1/review-options` | `reviews:write` | Your site's peer review options: `id`, `label`, and whether a comment is required. |
| `GET /v1/review-tasks` | `reviews:write` | The reviewer's open tasks, newest first (`limit` 1–500, default 100): `task_id`, type (`peer_review` or `community_watch`), accession number and Study Instance UID. |
| `POST /v1/review-tasks/{task_id}/claim` | `reviews:write` | Locks the case for this reviewer while they work. Call it again periodically while the case is open. |
| `POST /v1/review-tasks/{task_id}/determination` | `reviews:write` | Records the review. Body: `{"determination_id": <id from review-options>, "great_call": false, "technical_error": false, "comments": "…"}` |
| `POST /v1/review-tasks/{task_id}/defer` | `reviews:write` | Hands the task back; Guardian reassigns it. |
| `POST /v1/review-tasks/{task_id}/finalize` | `reviews:finalize` | Champion final decision on a reviewed case (same body as determination). The reviewer named must be a champion. The first final decision wins; a second one is refused (409). |

After a review is recorded, Guardian sends the same notifications as for a review done in its own viewer.

### Patient data

Without the `phi:read` scope, responses carry no patient-identifying details or report text: only identifiers your systems already hold (accession number, Study / Series / SOP Instance UIDs), codes and statuses.

With `phi:read`, responses also include the report text and sections, patient age, the report excerpts behind each finding, and critical-finding descriptions. Every such response is recorded in your audit log (key, accessions, source address) **before** it is sent; if the access can't be recorded, the request fails rather than returning the data.

Give `phi:read` only to keys that need it.

### Errors

| Status | Meaning |
|--------|---------|
| 401 | Missing or invalid key, or a key sent in the URL |
| 403 | The API is off for your site, the caller's address isn't on the key's allow-list, the key lacks the scope, or (finalize) the reviewer isn't a champion |
| 404 | No such exam, or no such task for this reviewer |
| 409 | The task was already finalized |
| 422 | Missing or unknown reviewer, or an invalid parameter |
| 429 | Rate limit reached, or too many failed attempts; try again later |
| 503 | Guardian couldn't record a patient-data access; try again |

Responses are JSON and are sent with `Cache-Control: no-store`.

---

## PACS Flag Case button

### Flag Case deep link

A button in your PACS opens this link in the radiologist's browser. Guardian finds the study, identifies the radiologist and opens the **Flag Case** tab of the Guardian dashboard, where they flag the case for peer review.

**Endpoint**

```
GET https://<your Guardian address>/flag-case?accession={accession}&user_name={user_name}
```

**Parameters**

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `accession` | string | Accession number of the study to flag. Without it, the radiologist can look the study up after opening the page. | No (recommended) |
| `user_name` | string | Who is flagging: their work email, their reporting-system user ID, or their exact full name. Without it, the radiologist signs in. | No (recommended) |

**Example**

```
https://guardian.yourhospital.com/flag-case?accession=ACC123456&user_name=drsmith
```

**Security**

- The button works without a separate sign-in **only from your organization's registered network ranges** (your web access ranges, set with Zauron). No button link works while no ranges are set, and requests from other addresses must sign in.
- A button click opens that one case only. It doesn't sign the radiologist in to the rest of Guardian.
- Requests are rate-limited.
- Guardian can be shown inside another application's page only when that site is on your embedding list.

**PACS button configuration**

Configure your PACS to add a custom button that:
1. reads the current study's accession number;
2. reads the current user's reporting-system ID or work email;
3. opens `https://<your Guardian address>/flag-case?accession={accession}&user_name={user_name}` in the default web browser.

**User experience**

When a radiologist clicks the button in their PACS:
1. their browser opens a new tab;
2. the Guardian Flag Case tab opens with the study selected;
3. they flag the case and add notes;
4. they return to their PACS workflow.

**Flag Case view**

Zauron sets, for each customer, what the Flag Case tab shows:

| View | Best for | The radiologist sees |
|------|----------|----------------------|
| **Full report** | Flagging from PACS or the Guardian dashboard, where the report isn't already open beside the button | The full report text and the peer review options. They can select text in the report to quote it into their comments, marked as a great call or a possible concern. They can also switch to another of the patient's studies if the wrong one was flagged. |
| **Peer review options only** | A Flag Case button inside your reporting software, where the radiologist already has the report open | Only the peer review determination, Great Call, Technical Issue and comments. Guardian doesn't send the report text or the patient's other studies to the browser. |

Tell Zauron which view you want when you set up the button. You can change it at any time.

---

### PACS worklist event sync (retired)

Bidirectional PACS worklist event sync (`/api/v1/pacs/events/*`) was **retired in April 2026** and is no longer part of Guardian Pro.

Use DICOM connectivity (C-FIND / C-MOVE / C-STORE) for study access, and the **Flag Case** deep link above for radiologist-initiated peer review from PACS. Optional worklist assignment for unreported orders is configured on the Guardian dashboard when enabled for your site — it is not this event-sync API.

### Testing the button

From a computer inside your registered network ranges, open the link in a browser:

```
https://guardian.yourhospital.com/flag-case?accession=ACC123456&user_name=drsmith
```
