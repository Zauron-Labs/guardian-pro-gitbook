# API Reference

This document covers two distinct API systems within Guardian Pro:

1. **Guardian Model API** — for integrating custom AI models with Guardian Pro
2. **Guardian Pro PACS Integration API** — for flagging cases from PACS via a browser deep link

---

# Guardian Model API

**Version 1.2** | API specification for Guardian-compatible model containers.

## Overview

Guardian runs your model in one of two ways:

- **Container:** Guardian pulls your model image from Zauron's container registry and runs it next to Guardian.
- **Hosted endpoint:** your model runs as an HTTPS service that Guardian calls.

Either way, your model receives one DICOM study at a time as a **gzip-compressed TAR archive** (`.tar.gz`) and returns JSON predictions. A container listens on port **8000**.

| Endpoint | Required | Used for |
|----------|----------|----------|
| `GET /icd10-mapping` | **Yes** | Readiness. Guardian calls it until it returns `200` before sending studies. Its mappings name the findings the model reports. |
| `POST /predict` | **Yes** | Inference on one study |
| `GET /health-check` | Hosted endpoints with async predict | Tells Guardian that `POST /predict/async` is available |
| `POST /predict/async` + `GET /predict/jobs/{job_id}` | Optional, hosted endpoints only | Long-running inference (for example large CT studies) |

Containers always receive synchronous `POST /predict` calls. Container images should include `curl`, which the container health check uses to call `GET /icd10-mapping`.

**Authentication (hosted endpoints):** Guardian sends your endpoint's API key in the `X-API-Key` header. Tell Zauron if your endpoint expects `Authorization: Bearer <key>` instead.

---

## Input Format

| Field | Requirement |
|-------|-------------|
| **Format** | gzip-compressed TAR archive (`.tar.gz`) of raw DICOM files |
| **File types** | `.dcm`, `.dicom`, or extensionless files with a DICM header |

Your model handles all processing internally (windowing, normalization, inference). Guardian always sends `.tar.gz`. Accepting an uncompressed `.tar` as well makes local testing easier.

---

## Endpoints

### `GET /icd10-mapping`

Returns the findings your model reports, mapped to ICD-10 codes. Guardian polls this endpoint until it returns `200`, and records each mapping's `code` and `description`.

**Response** `200 OK`
```json
{
  "mappings": {
    "pulmonary_embolism": {
      "code": "I26.9",
      "description": "Pulmonary embolism without acute cor pulmonale"
    }
  },
  "count": 1,
  "last_updated": "startup"
}
```

---

### `POST /predict`

Processes one DICOM study and returns predictions.

**Request**
```
Content-Type: multipart/form-data

study: <tar.gz file>           # Required
conf_threshold: 0.2            # Always sent, float 0.0-1.0: return findings at or above this confidence
mask_threshold: 0.5            # Sent only when the site sets one (segmentation models), float 0.0-1.0
```

**Response** `200 OK`
```json
{
  "generated_report": "Clinical findings summary.",
  "predictions": [
    {
      "icd10_code": "I26.9",
      "confidence": 0.95,
      "bounding_box": [0.45, 0.32, 0.12, 0.08],
      "study_uid": "1.2.840.113619.2.55.3.123456",
      "series_uid": "1.2.840.113619.2.55.3.123456.1",
      "instance_uid": "1.2.840.113619.2.55.3.123456.1.1",
      "abnormality": "pulmonary_embolism"
    }
  ]
}
```

Return an empty `predictions` list when nothing is found.

#### Prediction Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `icd10_code` | string | **Yes** | ICD-10 code of the finding, for example `I26.9`. Predictions without a valid ICD-10 code are ignored. |
| `confidence` | float | Recommended | 0-1. `conf` is accepted as an alias. |
| `bounding_box` | array[4] | No | `[x_center, y_center, width, height]`, normalized 0-1 (YOLO format) |
| `series_uid` | string | No | DICOM SeriesInstanceUID of the image with the finding |
| `instance_uid` | string | No | DICOM SOPInstanceUID of the image with the finding |
| `study_uid` | string | No | DICOM StudyInstanceUID. Guardian matches results to the study it sent, so this is informational. |
| `abnormality` | string | No | Finding name, for your own logs |

`generated_report` (string, optional) is a free-text summary of the study.

---

### `GET /health-check`

Needed only by hosted endpoints that implement async predict.

**Response** `200 OK`
```json
{
  "status": "healthy",
  "model_loaded": true,
  "predict_api": {
    "sync": true,
    "async": true
  }
}
```

Set `predict_api.async: true` when `POST /predict/async` is implemented. Omit `predict_api`, or set `async` to `false`, for sync-only endpoints.

---

### `POST /predict/async`

Optional, hosted endpoints only. Use it when inference may run longer than a proxy's idle timeout. It takes the same multipart fields as `POST /predict`.

**Response** `202 Accepted` with a non-empty `job_id`:
```json
{
  "job_id": "uuid",
  "status": "queued"
}
```

### `GET /predict/jobs/{job_id}`

Guardian polls this until `status` is `complete` or `failed`. Return `queued` or `running` while the job is in progress.

**Response** `200 OK` (complete): the same payload as `POST /predict`, plus the job fields.
```json
{
  "job_id": "uuid",
  "status": "complete",
  "generated_report": "Clinical findings summary.",
  "predictions": []
}
```

**Response** `200 OK` (failed): put the reason in `error` (or `message`).
```json
{
  "job_id": "uuid",
  "status": "failed",
  "error": "Series could not be decoded"
}
```

A `404` for a job Guardian submitted ends that prediction as failed.

Guardian allows a predict call (sync, or async submit plus polling) **30 minutes** by default.

---

## Error Responses

**`400 Bad Request`**
```json
{
  "detail": "Invalid input: TAR file contains no valid DICOM files"
}
```

**`500 Internal Server Error`**
```json
{
  "detail": "Internal server error: Model inference failed"
}
```

---

## Performance Requirements

| Metric | Requirement |
|--------|-------------|
| Inference time | Complete within the predict timeout (default **30 minutes**). Hosted endpoints can use async predict for long-running studies. |
| Memory | Size the container to the model; 8 GB is a common minimum for 2D models |
| Readiness | `GET /icd10-mapping` answers within 5 seconds once the model is loaded |

---

# Guardian Pro PACS Integration API

## Overview

Guardian Pro integrates directly with your PACS system, allowing radiologists to flag cases for peer review in real-time through custom PACS buttons or workflows. When clicked, the button opens a web browser and navigates to the Guardian Pro interface.

## Button Manual Trigger

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

---

## PACS Worklist Bidirectional Event Sync (retired)

Bidirectional PACS worklist event sync (`/api/v1/pacs/events/*`) was **retired in April 2026** and is no longer part of Guardian Pro.

Use DICOM connectivity (C-FIND / C-MOVE / C-STORE) for study access, and the **Flag Case** deep link above for radiologist-initiated peer review from PACS. Optional worklist assignment for unreported orders is configured on the Guardian dashboard when enabled for your site — it is not this event-sync API.

---

## Guardian Model API Testing

```bash
# Readiness and mappings
curl http://localhost:8000/icd10-mapping

# Synchronous prediction (.tar.gz)
curl -X POST http://localhost:8000/predict \
  -F "study=@study.tar.gz" \
  -F "conf_threshold=0.3"

# Asynchronous prediction (hosted endpoints that advertise it on /health-check)
curl http://localhost:8000/health-check
curl -X POST http://localhost:8000/predict/async \
  -F "study=@study.tar.gz" \
  -F "conf_threshold=0.3"
curl http://localhost:8000/predict/jobs/<job_id>
```

---

## Guardian Pro PACS Integration Testing

From a computer inside your registered network ranges, open the link in a browser:

```
https://guardian.yourhospital.com/flag-case?accession=ACC123456&user_name=drsmith
```
