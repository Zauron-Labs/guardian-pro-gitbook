# AI Models

Guardian Pro is model-neutral. It runs AI imaging models from different sources through one integration contract. Guardian uses each model's findings to pick the cases most likely to hold a quality concern.

## Open-source and proprietary models

Guardian supports both:

- **Open-source models**, packaged and validated for use with Guardian.
- **Proprietary models**, including commercial products from AI vendors and models your organization has built.

Every model meets the same [Guardian Model API](api_reference.md), so a model can be added, replaced or compared without changing your workflow. Integrating your own model is a no-cost add-on.

## Supported use cases

The use cases below have a supported model integration today. Which model serves a use case at your site, and whether it is open-source or proprietary, depends on your agreement.

| Use case | Modality | Body part |
| --- | --- | --- |
| Abdominal and thoracic aortic aneurysm | CT | Abdomen, chest |
| Brain MRI findings: hemorrhage, infarct, tumor | MR | Head |
| Breast cancer | MG | Breast |
| Cardiomegaly, pleural effusion, pneumonia | XR | Chest |
| Cirrhosis | CT | Abdomen |
| Coronary artery calcium | CT | Chest |
| Extremity fracture | XR | Extremities |
| Intracranial hemorrhage | CT | Head |
| Pneumothorax | XR | Chest |
| Pulmonary embolism | CT | Chest |
| Pulmonary nodule | CT | Chest |
| Renal cell carcinoma | CT | Abdomen, pelvis |
| Rib fracture | CT | Chest |
| Vertebral fracture | XR | Chest, spine |

Need a use case that isn't listed? Contact your Zauron representative. New integrations are added regularly.

## Choosing and tuning models

- **Licensing:** models are licensed per customer. Nothing is enabled until the models in your agreement are turned on for your organization.
- **Clinical rules:** your site decides how each licensed model is used, in the dashboard **Model Zoo**: which procedures trigger it (modality, body part, CPT), which findings count, and its confidence thresholds.
- **Operating thresholds:** each use case runs at a threshold chosen so the cases sent to reviewers are worth their time. Your site can adjust it, and the peer review options set how many AI-selected cases reach each radiologist (see [Options](options.md)).
- **Measure before you rely on it:** from the Model Zoo, a site admin or champion can start a [structured assessment](dataforge/overview.md#model-validation-and-structured-assessments) to measure a model against radiologist ground truth on your own studies.

## How models run

Guardian runs each model either as a container next to Guardian or as a hosted endpoint in the same cloud region as your Guardian site. See the [Guardian Model API](api_reference.md) for the integration contract.
