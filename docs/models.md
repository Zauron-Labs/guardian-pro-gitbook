# AI Models

Guardian Pro is compatible with purpose-built AI models to identify quality-relevant findings across a range of imaging modalities and body parts.

**Minimum Positive Predictive Value (PPV)** represents the lowest acceptable precision threshold validated on clinical data. Models at or above this threshold are deployed for case selection.

## How models are delivered

Models are **licensed per customer**. A new customer has no models enabled until Zauron enables the ones in its agreement. Your site then decides how each licensed model is used, in the dashboard **Model Zoo**: which procedures trigger it (modality, body part, CPT), its confidence thresholds and which findings count.

Guardian runs each model either as a container next to Guardian or as a hosted endpoint in the same cloud region as your Guardian site.

From the Model Zoo, a site admin or champion can also start a [structured assessment](dataforge/overview.md#model-validation-and-structured-assessments) to measure a model on your own studies.

Custom models use the same contract. See the [Guardian Model API](api_reference.md).

## Model Catalog (Alphabetical by Abnormality Type)

| Abnormality Type | Modality | Body Part | Minimum Positive Predictive Value | Open or Closed Source |
| --- | --- | --- | --- | --- |
| Breast Cancer | MG | Breast | 0.25 | Open |
| Cardiomegaly, Pleural Effusion, Pneumonia | XR | Chest | 0.75 | Open |
| Coronary Artery Calcium | CT | Chest | 0.75 | Open |
| Disc degeneration | MR | Lumbar Spine | 0.75 | Closed |
| Intracranial hemorrhage | CT | Head | 0.25 | Open |
| Ischemia | MR | Head | 0.25 | Closed |
| Pneumothorax | XR | Chest | 0.25 | Open |
| Pulmonary embolism | CT | Chest | 0.25 | Open |
| Vertebral fracture | XR | Chest | 0.25 | Open |

## Quick Filters

### By Modality

#### CT

| Abnormality Type | Body Part | Minimum Positive Predictive Value | Open or Closed Source |
| --- | --- | --- | --- |
| Coronary Artery Calcium | Chest | 0.75 | Open |
| Intracranial hemorrhage | Head | 0.25 | Open |
| Pulmonary embolism | Chest | 0.25 | Open |

#### MG

| Abnormality Type | Body Part | Minimum Positive Predictive Value | Open or Closed Source |
| --- | --- | --- | --- |
| Breast Cancer | Breast | 0.25 | Open |

#### MR

| Abnormality Type | Body Part | Minimum Positive Predictive Value | Open or Closed Source |
| --- | --- | --- | --- |
| Disc degeneration | Lumbar Spine | 0.75 | Closed |
| Ischemia | Head | 0.25 | Closed |

#### XR

| Abnormality Type | Body Part | Minimum Positive Predictive Value | Open or Closed Source |
| --- | --- | --- | --- |
| Cardiomegaly, Pleural Effusion, Pneumonia | Chest | 0.75 | Open |
| Pneumothorax | Chest | 0.25 | Open |
| Vertebral fracture | Chest | 0.25 | Open |

### By Source

#### Open

| Abnormality Type | Modality | Body Part | Minimum Positive Predictive Value |
| --- | --- | --- | --- |
| Breast Cancer | MG | Breast | 0.25 |
| Cardiomegaly, Pleural Effusion, Pneumonia | XR | Chest | 0.75 |
| Coronary Artery Calcium | CT | Chest | 0.75 |
| Intracranial hemorrhage | CT | Head | 0.25 |
| Pneumothorax | XR | Chest | 0.25 |
| Pulmonary embolism | CT | Chest | 0.25 |
| Vertebral fracture | XR | Chest | 0.25 |

#### Closed

| Abnormality Type | Modality | Body Part | Minimum Positive Predictive Value |
| --- | --- | --- | --- |
| Disc degeneration | MR | Lumbar Spine | 0.75 |
| Ischemia | MR | Head | 0.25 |

## Search Tips

- Use GitBook's built-in page search to find values in any column (e.g., `Chest`, `0.75`, or `Closed`).
- Use your browser's in-page find (`Ctrl+F` / `Cmd+F`) for quick column-level lookup.
