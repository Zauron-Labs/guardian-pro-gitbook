# API Reference

This document covers two distinct API systems within Guardian Pro:

1. **Guardian Model API** — for integrating custom AI models with Guardian Pro
2. **Guardian Pro PACS Integration API** — for flagging cases from PACS via a browser deep link

---

# Guardian Model API

**Version 1.1** | API specification for Guardian-compatible model containers.

## Overview

Guardian pulls model images from the Zauron Azure Container Registry and runs them as modular containers (or as remote HTTP endpoints). Your container receives DICOM studies as **gzip-compressed TAR archives** (`.tar.gz`) and returns JSON predictions. Expose port **8000** with the endpoints below.

Synchronous `POST /predict` is required. If inference may exceed a few minutes (common for large CT studies), also implement async predict. Guardian discovers async support from `GET /health-check` and will use `POST /predict/async` automatically.

---

## Input Format

| Field | Requirement |
|-------|-------------|
| **Format** | gzip-compressed TAR archive (`.tar.gz`) containing raw DICOM files |
| **File types** | `.dcm`, `.dicom`, or extensionless with DICM header |
| **Max size** | 8GB |

Your container handles all processing internally (windowing, normalization, inference). Guardian still accepts uncompressed `.tar` for compatibility; production traffic is `.tar.gz`.

---

## Endpoints

### `GET /health-check`

Returns service status.

**Response** `200 OK`
```json
{
  "status": "healthy",
  "model_loaded": true,
  "message": "Service ready for predictions",
  "predict_api": {
    "sync": true,
    "async": true
  }
}
```

Include `predict_api.async: true` when `POST /predict/async` is implemented. Guardian uses this field to choose sync vs async at runtime. Omit `predict_api` (or set `async` to `false`) for sync-only models.

**Response** `500` (if unhealthy)
```json
{
  "status": "unhealthy",
  "model_loaded": false,
  "message": "Model failed to load"
}
```

---

### `POST /predict`

Processes a DICOM study and returns predictions.

**Request**
```
Content-Type: multipart/form-data

study: <tar.gz file>           # Required
conf_threshold: 0.2            # Optional, float 0.0-1.0
```

**Response** `200 OK`
```json
{
  "generated_report": "Clinical findings summary.",
  "predictions": [
    {
      "abnormality": "pulmonary_embolism",
      "bounding_box": [0.45, 0.32, 0.12, 0.08],
      "confidence": 0.95,
      "study_uid": "1.2.840.113619.2.55.3.123456",
      "series_uid": "1.2.840.113619.2.55.3.123456.1",
      "instance_uid": "1.2.840.113619.2.55.3.123456.1.1",
      "icd10_code": "I26.9",
      "icd10_description": "Pulmonary embolism without acute cor pulmonale",
      "icd10_snippet": "PE detected. Recommend anticoagulation therapy."
    }
  ]
}
```

#### Prediction Fields

| Field | Type | Description |
|-------|------|-------------|
| `abnormality` | string | Condition name |
| `bounding_box` | array[4] | YOLO format `[x, y, w, h]`, normalized 0-1 |
| `confidence` | float | 0-1 |
| `study_uid` | string | DICOM StudyInstanceUID |
| `series_uid` | string | DICOM SeriesInstanceUID |
| `instance_uid` | string | DICOM SOPInstanceUID |
| `icd10_code` | string | ICD-10 diagnosis code |
| `icd10_description` | string | ICD-10 description |
| `icd10_snippet` | string | Clinical context snippet |

All fields are required.

---

### `POST /predict/async`

Optional. Use when inference may run longer than a reverse-proxy idle timeout. Same multipart fields as `POST /predict`.

**Response** `202 Accepted`
```json
{
  "job_id": "uuid",
  "status": "queued",
  "study_filename": "study.tar.gz",
  "conf_threshold": 0.2,
  "submitted_at": "2026-06-14T12:00:00+00:00",
  "poll_url": "/predict/jobs/{job_id}",
  "message": "Waiting for inference"
}
```

### `GET /predict/jobs/{job_id}`

Poll until `status` is `complete` or `failed`. While queued or running, return those statuses. On success, return the same prediction payload as `POST /predict`, plus job metadata.

**Response** `200 OK` (complete)
```json
{
  "job_id": "uuid",
  "status": "complete",
  "generated_report": "Clinical findings summary.",
  "predictions": []
}
```

Guardian's wall-clock budget for a predict call (sync HTTP timeout, or async submit-plus-poll) defaults to **1800 seconds** (`GUARDIAN_CV_PREDICT_TIMEOUT_SEC`).

---

### `GET /icd10-mapping`

Returns current ICD-10 mappings.

**Response** `200 OK`
```json
{
  "mappings": {
    "pulmonary_embolism": {
      "code": "I26.9",
      "description": "Pulmonary embolism without acute cor pulmonale",
      "icd10_snippet": "PE detected. Recommend anticoagulation therapy."
    }
  },
  "count": 1,
  "last_updated": "startup"
}
```

---

### `POST /icd10-mapping`

Updates ICD-10 mappings.

**Request**
```json
{
  "pulmonary_embolism": {
    "code": "I26.9",
    "description": "Pulmonary embolism without acute cor pulmonale",
    "icd10_snippet": "PE detected. Recommend anticoagulation therapy."
  }
}
```

**Response** `200 OK`
```json
{
  "status": "updated",
  "mappings_count": 1,
  "message": "ICD-10 mappings updated successfully"
}
```

---

### `GET /logging`

Returns container logs.

**Response** `200 OK`
```json
{
  "logs": [
    "2024-01-15 10:00:00 INFO: Model loaded successfully",
    "2024-01-15 10:01:23 INFO: Processing study..."
  ],
  "log_count": 2,
  "timestamp": "2024-01-15T10:01:30Z"
}
```

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
| Inference time | Complete within the site predict timeout (default **30 minutes**). Use async predict for long-running studies. |
| Memory | Size the container to the model; 8 GB is a common minimum for 2D models |
| Health check | < 5 seconds |

---

# Guardian Pro PACS Integration API

## Overview

Guardian Pro integrates directly with your PACS system, allowing radiologists to flag cases for peer review in real-time through custom PACS buttons or workflows. When clicked, the button opens a web browser and navigates to the Guardian Pro interface.

## Button Manual Trigger

### Flag Case API Endpoints

#### Deep Link — Create

This endpoint creates a flagged case and redirects the user to the Guardian Pro interface.

**Endpoint**

Behind Traefik (TLS): `https://<GUARDIAN_DOMAIN>`

```
GET /flag-case?accession={accession}&user_name={user_name}
```

**Parameters**

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `accession` | string | Accession number of the study to flag | Yes |
| `user_name` | string | Username or identifier of the radiologist flagging the case | Yes |

**Example**

```
GET https://guardian.yourhospital.com/flag-case?accession=ACC123456&user_name=drsmith
```

**Response**

The endpoint opens in a web browser and redirects the user to the Guardian Pro viewer where they can:
- Review the flagged study
- Add comments or notes
- Submit the case for peer review

**PACS Button Configuration**

Configure your PACS to add a custom button with the following behavior:
1. Extract the current study's accession number
2. Extract the current user's username
3. Open URL in default web browser: `https://<GUARDIAN_DOMAIN>/flag-case?accession={accession}&user_name={user_name}`

**User Experience**

When a radiologist clicks the button in their PACS application:
1. Their default web browser opens (or a new tab if browser is already open)
2. Browser navigates to the Guardian Pro interface with the study pre-loaded
3. Radiologist can flag the case and add relevant notes
4. Radiologist can return to their PACS workflow

---

## PACS Worklist Bidirectional Event Sync (retired)

Bidirectional PACS worklist event sync (`/api/v1/pacs/events/*`) was **retired in April 2026** and is no longer part of Guardian Pro.

Use DICOM connectivity (C-FIND / C-MOVE / C-STORE) for study access, and the **Flag Case** deep link above for radiologist-initiated peer review from PACS. Optional worklist assignment for unreported orders is configured on the Guardian dashboard when enabled for your site — it is not this event-sync API.

---

## Guardian Model API Testing

```bash
# Health check
curl http://localhost:8000/health-check

# Synchronous prediction (.tar.gz)
curl -X POST http://localhost:8000/predict \
  -F "study=@study.tar.gz" \
  -F "conf_threshold=0.3"

# Asynchronous prediction (when advertised on /health-check)
curl -X POST http://localhost:8000/predict/async \
  -F "study=@study.tar.gz" \
  -F "conf_threshold=0.3"

# Get mappings
curl http://localhost:8000/icd10-mapping

# Logs
curl http://localhost:8000/logging
```

---

## Guardian Pro PACS Integration Testing

```bash
# Test flag case endpoint (replace with your domain and parameters)
curl "https://guardian.yourhospital.com/flag-case?accession=ACC123456&user_name=drsmith"
```

