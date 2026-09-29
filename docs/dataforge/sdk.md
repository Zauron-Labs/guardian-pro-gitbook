# DataForge Python SDK

## Overview

The DataForge Python SDK lets you work with one DataForge **labeling project** from your own code: upload images, import reports and AI predictions, set the label contract, assign work, follow progress, read labels, import ground truth, download images and export results.

It authenticates with a **project API key**. A key belongs to exactly one labeling project and can act only on that project.

The package is `dataforge-sdk` (import name `dataforge_sdk`). It also installs a `dataforge` command for shell scripts.

## Requirements

- Python 3.9 or newer. No other packages are needed.
- Your workstation can reach the DataForge address over HTTPS, for example `https://<host>/dataforge`. This is usually from inside your organization's network or VPN.
- If your organization uses a private certificate authority, you need its CA certificate as a PEM file.
- A project API key for the labeling project (see the next section).

## Get an API key

1. Open the labeling project in DataForge and go to **Project Settings → API keys**. Only project admins see this section, and only labeling projects have API keys.
2. Choose **Create key**, give the key a name that says what it is for (for example `nodule-import-pipeline`), and choose how long it lasts: 30, 90, 180 or 365 days.
3. Copy the key right away. It starts with `dfk_` and is **shown only once**. DataForge stores only a fingerprint of it, so a lost key cannot be recovered. Create a new key instead.
4. To stop a key, choose **Revoke** in the same section. It stops working immediately. Expired and revoked keys stay listed so you can see when each was last used.

Keep the key out of your code. Put it in an environment variable:

```bash
export DATAFORGE_API=https://<host>/dataforge
export DATAFORGE_API_KEY=dfk_...
```

## Admin keys

An **admin key** is for automation that sets projects up: creating labeling projects, configuring them and issuing researcher keys. It covers every project of your DataForge site, and only that site.

**Where they are issued.** An administrator creates admin keys in the main application: **Admin → Settings → DataForge module API keys**. As with project keys:

- the admin key starts with `dfa_` and is shown only once;
- it lasts 30, 90, 180 or 365 days;
- it can be revoked there at any time.

Put it in `DATAFORGE_ADMIN_API_KEY`.

**What an admin key can do:**

- List, create and update projects. It can create `labeling` or `model_validation` projects and change a project's name, description and settings.
- Do everything a project key can, on any project: labels, images, reports, predictions, tasks, exports and image download.
- Manage a project's image sources and members, including making a member a project admin.
- Issue, list and revoke project (researcher) keys.

**What it cannot do:**

- delete or purge projects;
- create Model Zoo assessments, or change a locked assessment's evidence;
- query or retrieve from PACS, or run models;
- see privacy exclusions or re-identification data, or grant privacy officer rights;
- manage admin keys.

**Worked example: create a labeling project, set it up and hand a researcher a key.**

```python
from dataforge_sdk import DataForgeAdmin, Label

admin = DataForgeAdmin()                      # DATAFORGE_API + DATAFORGE_ADMIN_API_KEY
project = admin.projects.create(
    "Lung nodules 2026-Q4",
    workflow_type="labeling",
    description="Chest CT nodule boxes",
    labels=[Label(code="nodule", display_name="Lung nodule", kind="bbox")],
)

p = admin.project(project.id)                 # same methods as DataForgeClient
p.upload_images(preprocessed_dir="/data/preprocessed/lung_q4", anonymize=True, wait=True)
p.add_member(1234, role="annotator")

key = admin.api_keys(project.id).create("alice-research", expires_in_days=90)
print("Give this key to the researcher (shown once):", key.key)
```

The same steps from the shell:

```bash
dataforge projects create "Lung nodules 2026-Q4" --labels labels.json
dataforge --project 42 ingest --preprocessed-dir /data/preprocessed/lung_q4 --anonymize --wait
dataforge keys create --project 42 --name alice-research
```

## Install

Install from your DataForge server (no internet access needed):

```bash
pip install --find-links https://<host>/dataforge/sdk/ dataforge-sdk
```

To install on a machine that cannot reach the server, first download the wheel file linked at `https://<host>/dataforge/sdk/`, for example `dataforge_sdk-0.1.0-py3-none-any.whl`. Copy it across, then run:

```bash
pip install ./dataforge_sdk-0.1.0-py3-none-any.whl
```

If the server uses a private certificate authority, add `--cert /path/to/ca.pem` to the `pip` command.

Check the installation:

```bash
python -c "import dataforge_sdk; print(dataforge_sdk.__version__)"
dataforge whoami
```

## Quickstart

This example builds a labeling project end to end. Each step is optional; run the ones you need.

```python
from dataforge_sdk import DataForgeClient, Label, AnnotationInput

client = DataForgeClient()            # reads DATAFORGE_API and DATAFORGE_API_KEY
print(client.whoami())                # key name, expiry and the project it works on
print(client.project().name)

# 1. Upload images.
#    Preferred: a folder that is already on the DataForge server (no upload).
job = client.upload_images(preprocessed_dir="/data/preprocessed/cohort_2026_09",
                           anonymize=True, wait=True)
#    Or stream a local archive from this workstation:
# job = client.upload_images(file="cohort.tar.gz", wait=True)
print(job.status, job.processed_studies, "studies")

# 2. Import radiology reports (for an anonymized upload, pass its job id).
client.import_reports("reports.csv", ingest_job_id=job.id)

# 3. Set the label contract.
client.labels.set([
    Label(code="nodule", display_name="Lung nodule", kind="bbox"),
    Label(code="no_finding", display_name="No finding", kind="study_level"),
])

# 4. Import AI predictions and turn them into editable prelabels.
client.import_predictions("predictions.csv", materialize_prelabels=True)

# 5. Assign studies to annotators (employee ids of project members).
studies = [s.study_event_id for s in client.studies() if s.has_imaging and not s.tasks]
client.assign(studies[:50], assignee=1234, role="annotator")

# 6. Monitor progress.
p = client.progress()
print(f"{p.completed_tasks}/{p.total_tasks} tasks complete ({p.percent_complete}%)")

# 7. Read labels, or import your own ground truth.
for ann in client.annotations(studies[0]):
    print(ann.label_code, ann.label_kind, ann.bounding_box)
client.import_annotations([
    AnnotationInput(study_event_id=studies[1], label_code="no_finding", label_kind="study_level"),
])

# 8. Export results and download images.
client.export("annotation-packet", "packet.zip")
client.download_study_images(studies[0], "./images")
```

## Concepts

**Project.** A labeling project in DataForge. A project API key is bound to one project: `client.project_id` is that project, and every call acts on it.

**Key types.** There are two kinds of key:

- A **project key** (`dfk_…`, used with `DataForgeClient`) is created in a project's Settings and works on that one project. It is meant for researchers.
- An **admin key** (`dfa_…`, used with `DataForgeAdmin`) is created in the main application under Admin → Settings → DataForge module API keys and works across your site's projects. It is meant for automation that creates and configures projects. See [Admin keys](#admin-keys).

**Studies.** An imaging exam in the project, identified by a *study id*. The study id is the Study Instance UID, or the id assigned during upload for anonymized data. `studies()` lists every study of the project with its imaging and task state; `imaging_available()` lists the ones whose images are ready.

**Label contract.** The labels annotators may use. Each label has a `code` and one `kind`:

| Kind | Meaning | Location fields |
|------|---------|-----------------|
| `bbox` | A box on one image | `series_uid`, `instance_uid`, `bounding_box` |
| `study_level` | A classification of the whole exam | none |
| `series_level` | A classification of one series | `series_uid` |
| `image_level` | A classification of one image | `series_uid`, `instance_uid` |

A `bounding_box` is `[x, y, width, height]`, measured as fractions (0 to 1) of the image, from the top-left corner.

**Tasks, roles and status.** A task assigns one study to one project member. The member's role is `annotator` (draws labels) or `reviewer` (agrees or disagrees and finalizes). A task's status moves through `assigned` → `in_progress` → `prelim` → `final`, with `changes_requested` when a reviewer sends work back. Once a reviewer finalizes a study, its labels are locked.

**Prelabels.** AI predictions that match the label contract can be copied into editable labels (`origin = "ai_prelabel"`) for annotators to correct, instead of drawing from scratch.

**Tranches.** Groups of studies by AI confidence, used to sample review work across the confidence range. `tranches(recalculate=True)` draws a fresh sample.

## API reference

All methods raise the errors listed in [Error handling](#error-handling). Dates are ISO 8601 strings in UTC. Every result object has a `raw` dict with the complete server response.

### DataForgeClient(base_url=None, api_key=None, *, timeout=120, verify_tls=True, ca_bundle=None, max_retries=3, backoff=0.5)

Creates a client.

- `base_url` defaults to `DATAFORGE_API`, and `api_key` to `DATAFORGE_API_KEY`.
- `timeout` is in seconds per request.
- `ca_bundle` is a PEM file for a private certificate authority.
- `verify_tls=False` turns certificate checks off. Use it only for testing.
- Read calls are retried up to `max_retries` times on HTTP 429/502/503/504 and on network errors, with exponential backoff starting at `backoff` seconds.

Raises `ConfigurationError` if the URL or key is missing or malformed.

### Project

| Method | Returns | Notes |
|--------|---------|-------|
| `whoami()` | `KeyInfo` (`project_id`, `project_name`, `key_name`, `masked`, `expires_at`) | Checks that the key works. |
| `project_id` | `int` | The key's project. |
| `project()` | `Project` (`id`, `name`, `description`, `workflow_type`, `created_at`, `updated_at`) | |
| `config()` | `dict` | Project settings. |
| `update_config(changes)` | `dict` | Changes only the given settings. |

### Label contract

| Method | Returns | Notes |
|--------|---------|-------|
| `labels.get()` | `list[Label]` | |
| `labels.set(labels)` | `list[Label]` | Replaces the whole contract. `labels` is a list of `Label` objects or dicts with `code`, `display_name`, `kind`, `description`, `color_hex`, `hotkey_index`, `required` and `sort_order`. Raises `ConflictError` if someone else changed the contract at the same time; read it again and retry. |

### Images

**`upload_images(*, preprocessed_dir=None, archive=None, file=None, anonymize=False, wait=False, wait_timeout=7200, poll_interval=5)`** returns an `IngestJob`.

Give exactly one source:

- `preprocessed_dir`: a folder **on the DataForge server** containing `images/<study>/` and, optionally, `reports/<study>/`.
- `archive`: a `.tar`, `.tar.gz` or `.zip` **on the server**.
- `file`: a local archive, streamed from disk in 1 MiB chunks.

`anonymize=True` de-identifies the DICOM files. With `wait=True` the call returns when the job completes, and raises `IngestFailedError` if it fails.

Other image methods:

| Method | Returns | Notes |
|--------|---------|-------|
| `ingest_job(job_id)` | `IngestJob` (`id`, `status`, `phase`, `total_studies`, `processed_studies`, `failed_studies`, `anonymize`, `error`, `done`) | |
| `ingest_jobs(limit=50)` | `list[IngestJob]` | Newest first. |
| `wait_for_ingest(job_id, *, timeout=7200, poll_interval=5)` | `IngestJob` | |
| `studies()` | iterator of `Study` (`study_event_id`, `study_uid`, `accession_number`, `in_cohort`, `has_predictions`, `has_imaging`, `tasks`) | Every study of the project — its cohort, studies with AI predictions and studies with tasks. `tasks` lists each task's `task_id`, `role`, `status` and `assignee_employee_id`. |
| `imaging_available()` | `list[str]` | Identifiers of this project's studies with images ready (study ids and their Study Instance UIDs). |
| `study_instances(study_id)` | `dict` (`series`, `instances`) | Series and images of a study. |
| `download_study_images(study_id, dest_dir)` | `DownloadResult` (`study_dir`, `series`, `instances`, `frames`, `manifest_path`, `files`) | See [Image download format](#image-download-format). |

### Reports, predictions and cases

**`import_reports(csv_path, *, ingest_job_id=None)`** returns an `ImportResult`, with `raw` holding `succeeded_count`, `failed_count` and `failed_rows`.

- Upload images before importing their reports.
- Each CSV row needs `report_text` and one identifier: `study_event_id` / `StudyInstanceUID`, or `accession_number`. `radiologist_positive` is optional.
- For an anonymized upload, pass its `ingest_job_id`. The original identifiers in the CSV are then matched to the de-identified studies on the server.

**`import_predictions(path, *, materialize_prelabels=True, replace_existing=False)`** returns an `ImportResult`.

- `path` is a CSV or JSON file of AI detections.
- With `materialize_prelabels=True`, the detections that match the label contract become editable prelabels. `raw["prelabels"]` holds their counts.
- `replace_existing=True` removes the earlier prelabels first.

**`append_cases(body)`** returns a `dict`. It adds server-side data in one call. `body` may contain:

- `image_ingest`: `archive_path` or `preprocessed_root`, plus `anonymize`.
- `report_ingest`: `csv_path`.
- `prediction_ingest`: `file_path`, `materialize_prelabels`.

### Tranches, tasks and members

| Method | Returns | Notes |
|--------|---------|-------|
| `tranches(*, recalculate=False)` | `list[Tranche]` (`tranche_id`, `category`, `confidence_min`, `confidence_max`, `ppv`, `study_ids`) | |
| `tasks(*, status=None, role=None, assignee=None)` | `list[Task]` (`id`, `study_event_id`, `assignee_employee_id`, `assignee_email`, `assignee_name`, `role`, `status`, `assigned_at`, `completed_at`) | All tasks, optionally filtered. |
| `assign(study_ids, assignee, role="annotator")` | `dict` | Assigns studies to a project member. The assignee gets an email. |
| `members()` | `list[Member]` (`employee_id`, `email`, `name`, `role`, `settings_permission`) | |
| `add_member(employee_id, *, role=None)` | `Member` | Adds a member or sets their role (`annotator` / `reviewer`). |
| `progress()` | `Progress` (`total_tasks`, `completed_tasks`, `in_progress_tasks`, `assigned_tasks`, `percent_complete`) | Per-role and per-author detail is in `raw`. |
| `label_decisions()` | `list[dict]` | Per study and label code: `annotation_count`, `approved_count`, `rejected_count`. |

### Labels on studies

| Method | Returns | Notes |
|--------|---------|-------|
| `annotations(study_id)` | `list[Annotation]` (`id`, `study_event_id`, `label_code`, `label_kind`, `series_uid`, `instance_uid`, `bounding_box`, `origin`, `created_by_employee_id`, `created_at`) | |
| `annotation_reviews(study_id)` | `list[dict]` | Reviewer decisions on labels. |
| `segmentations(study_id)` | `list[dict]` | |
| `measurements(study_id)` | `list[dict]` | |
| `detection_reviews(study_id)` | `list[dict]` | Reviewer decisions on AI detections. |

**`import_annotations(annotations, *, skip_existing=True, batch_size=1000)`** returns an `AnnotationImportResult` (`created`, `skipped`, `failed`, `annotation_ids`).

- Imports ground-truth labels, given as `AnnotationInput` objects or dicts.
- Each label must use a code and kind from the label contract.
- These rows are refused and listed in `failed` (with `index` and `detail`): studies not in the project, studies a reviewer has finalized, and invalid rows.
- A label that already exists is counted in `skipped`.
- Imported labels are attributed to the person who created the API key. Existing labels are never changed or deleted.

### Exports

**`export(kind, path, *, gt_agreed_only=False)`** returns a `Path`. The file is streamed to `path`. `kind` is one of:

- `csv`: the results table.
- `annotation-packet`: a zip of labels, task audit and overlay images. `gt_agreed_only=True` keeps only labels that reviewers agreed with.
- `data-only`: a zip of images and reports.
- `qc-packet`: a zip.

### DataForgeAdmin(base_url=None, api_key=None, *, timeout=120, verify_tls=True, ca_bundle=None, max_retries=3, backoff=0.5)

The admin client. `api_key` defaults to `DATAFORGE_ADMIN_API_KEY` and must start with `dfa_`. The other options are the same as for `DataForgeClient`.

| Method | Returns | Notes |
|--------|---------|-------|
| `whoami()` | `KeyInfo` (`kind="admin"`, `key_name`, `masked`, `expires_at`) | |
| `projects.list()` | `list[Project]` | All projects of the site. |
| `projects.get(project_id)` | `Project` | |
| `projects.create(name, workflow_type="labeling", *, description="", labels=None, **settings)` | `Project` | `workflow_type` is `labeling` or `model_validation`. `labels` sets the label contract. Other keywords are project settings. |
| `projects.update(project_id, *, name=None, description=None, **settings)` | `Project` | The workflow type cannot change. |
| `project(project_id)` | `ProjectHandle` | Has every per-project method of `DataForgeClient` (labels, images, reports, tasks, annotations, export, download…), for that project. |
| `api_keys(project_id).list()` | `list[ApiKeyInfo]` (`id`, `name`, `masked`, `status`, `created_by_name`, `created_at`, `expires_at`, `last_used_at`, `revoked_at`) | Never includes secrets. |
| `api_keys(project_id).create(name, expires_in_days=90)` | `CreatedApiKey` (`id`, `name`, `masked`, `expires_at`, `key`) | `key` is the researcher key, shown only this once. Labeling projects only. |
| `api_keys(project_id).revoke(key_id)` | `None` | Takes effect immediately. |

### Image download format

`download_study_images(study_id, dest_dir)` writes:

```
<dest_dir>/<StudyInstanceUID>/
  manifest.json                       # series, instances and frame files with their content types
  <SeriesInstanceUID>/
    metadata.json                     # DICOM JSON model (PS3.18 Annex F) for the series' instances
    <SOPInstanceUID>/
      frame_0001.<ext>                # pixel data of frame 1, one file per frame
```

Frames are the stored pixel data, as a DICOMweb (WADO-RS) frame request returns it. The file extension shows the encoding:

- `.raw`: uncompressed native pixels, little endian. Rows, columns, bits allocated and photometric interpretation are in `metadata.json`.
- `.jls`: JPEG-LS.
- `.jp2`: JPEG 2000.
- `.jpg`: JPEG.
- `.jxl`: JPEG XL.
- `.png`: PNG.
- `.bin`: any other encoding.

Multi-frame images produce `frame_0001`, `frame_0002` and so on. Series or images that were excluded for privacy are not downloaded. Images that are still being prepared raise `NotFoundError`; retry later.

## CLI reference

The `dataforge` command (also `python -m dataforge_sdk`) reads `DATAFORGE_API` and `DATAFORGE_API_KEY`. The `projects` and `keys` commands read `DATAFORGE_ADMIN_API_KEY`. With an admin key, every project command also works on any project when you add `--project ID`, for example `dataforge --project 42 status`. Global options:

- `--api URL`
- `--api-key KEY` (prefer the environment variable, so the key does not appear in your shell history)
- `--admin-api-key KEY` (prefer `DATAFORGE_ADMIN_API_KEY`)
- `--project ID` (with an admin key: the project to work on)
- `--ca-bundle FILE`
- `--timeout SECONDS`

Output is JSON.

| Command | Does |
|---------|------|
| `dataforge whoami` | Key name, expiry and project. |
| `dataforge status` | Project, task progress, recent ingest jobs. |
| `dataforge labels get` / `dataforge labels set FILE` | Read or replace the label contract. `FILE` is a JSON list, inline JSON, or `-` for stdin. |
| `dataforge ingest (--preprocessed-dir DIR \| --archive PATH \| --upload FILE) [--anonymize] [--wait]` | Upload images. |
| `dataforge reports CSV [--ingest-job-id N]` | Import reports. |
| `dataforge append-cases FILE` | Append server-side data (JSON body). |
| `dataforge predictions FILE [--materialize-prelabels] [--replace-existing]` | Import AI predictions. |
| `dataforge tranches [--recalculate]` | Show or redraw tranches. |
| `dataforge tasks list [--status S] [--role R]` | List tasks. |
| `dataforge tasks assign --assignee ID [--role annotator\|reviewer] (--studies A,B \| --studies-file FILE)` | Assign studies. |
| `dataforge annotations get STUDY_ID` | Labels on a study. |
| `dataforge annotations import FILE [--no-skip-existing]` | Import ground truth. `FILE` is a JSON list of objects with `study_event_id`, `label_code`, `label_kind`, and location fields. |
| `dataforge download STUDY_ID -o DIR` | Download a study's images. |
| `dataforge export {csv,annotation-packet,data-only,qc-packet} -o FILE [--gt-agreed-only]` | Download an export. |
| `dataforge projects list` | Admin key: all projects. |
| `dataforge projects create NAME [--workflow labeling\|model_validation] [--description D] [--labels FILE]` | Admin key: create a project. |
| `dataforge projects update ID [--name N] [--description D]` | Admin key: rename or re-describe a project. |
| `dataforge keys list --project ID` | Admin key: a project's researcher keys. |
| `dataforge keys create --project ID --name N [--expires-in-days 30\|90\|180\|365]` | Admin key: issue a researcher key. The key is printed once. |
| `dataforge keys revoke --project ID KEY_ID` | Admin key: revoke a researcher key. |

Exit codes:

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Server or network error; the reason is printed |
| 2 | Usage error |
| 3 | Authentication failed (missing, invalid, revoked or expired key) |
| 4 | Permission denied |

## Error handling

| Exception | When |
|-----------|------|
| `ConfigurationError` | The base URL or key is missing or malformed, or the CA bundle file was not found. |
| `DataForgeConnectionError` | The server could not be reached: DNS, network, TLS or timeout. Read calls retry first. |
| `AuthError` (401) | The key is unknown, revoked or expired. Create a new key. |
| `PermissionDeniedError` (403) | Another project, or something API keys cannot do. |
| `NotFoundError` (404) | The study, job or file is not in this project, or images are not ready yet. |
| `ConflictError` (409) | A concurrent change or a duplicate. Read the current state and retry. |
| `ValidationError` (422) | The request content was rejected. `detail` says why. |
| `RateLimitError` (429) | Too many requests. Read calls retry automatically. |
| `ApiError` | Any other error status. All of the above HTTP errors are subclasses. |
| `IngestFailedError` | An upload job finished as failed. `error` has the reason. |

Every `ApiError` has `status` and `detail` (the server's reason):

```python
from dataforge_sdk import ApiError, AuthError

try:
    client.labels.set(labels)
except AuthError:
    print("The API key is no longer valid - create a new one in Project Settings.")
except ApiError as exc:
    print(exc.status, exc.detail)
```

## Security notes

- **Treat the key like a password.** Keep it in an environment variable or a secrets manager. Never commit it, paste it into tickets, or put it in a URL.
- **Admin keys are powerful.** An admin key reaches every project of your site. Keep it in a secrets manager for automation, never on a researcher's workstation. Give researchers project keys instead. Guardian administrators can see and revoke admin keys in Admin → Settings → DataForge module API keys.
- **Scope.** A project key works only on its own labeling project. It cannot:
  - create or delete projects;
  - manage API keys;
  - reach other projects;
  - see or change privacy exclusions or re-identification data;
  - delete labels that people drew.
- **Attribution.** Changes made with a key are recorded under the person who created the key, and the server audit log names the key.
- **Revoke** a key you no longer need, or one that may have leaked. Revoking takes effect immediately. Prefer short lifetimes, with one key per pipeline.
- **Header only.** The SDK sends the key only in the `Authorization` header. The server refuses keys passed in a URL.
- **TLS.** Certificates are verified by default. Use `ca_bundle` for a private certificate authority rather than turning verification off.
- **Downloaded images** follow the project's de-identification. Store them under your organization's data-handling rules.

## Limits

- One key belongs to one labeling project. A project can have up to 20 active keys. Keys last 30, 90, 180 or 365 days.
- A site can have up to 20 active admin keys.
- HTTP archive upload (`upload_images(file=...)`) is capped by the server, 512 MiB by default. Larger cohorts should be placed on the DataForge server and ingested with `preprocessed_dir` or `archive`.
- `import_annotations` sends at most 5,000 labels per request. The SDK splits larger lists into batches of `batch_size`.
- List calls return complete results. None of them page.
- Only images already prepared for viewing can be downloaded. A study still being prepared returns `NotFoundError` until it is ready.
