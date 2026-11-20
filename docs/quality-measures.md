# Quality Measures

Guardian Pro scores CMS MIPS quality measures on your final radiology reports. Results appear in the dashboard's **Quality** tab, and each measure exports for your MIPS registry.

## CMS MIPS measures

Guardian supports the CMS MIPS quality measures below for diagnostic radiology. For each exam, Guardian reads the final report to decide whether the exam belongs in the measure and whether the report documents what the measure asks for.

Each site starts with these measures turned off. A site admin or Quality Champion turns on the ones the site reports.

| Measure | Title | The report should document |
| --- | --- | --- |
| 360 | Count of potential high-dose radiation imaging studies: CT and cardiac nuclear medicine | A count of the patient's known CT and cardiac nuclear medicine studies in the 12 months before this study |
| 364 | Follow-up CT imaging for incidentally detected pulmonary nodules according to recommended guidelines | For an incidental lung nodule (patients 35 or older), a follow-up interval and modality, or no follow-up, with the named guideline source |
| 405 | Appropriate follow-up imaging for incidental abdominal lesions | For a simple-appearing renal cyst or a benign adrenal lesion (patients 18 or older), an explicit statement that no follow-up imaging is recommended |
| 406 | Appropriate follow-up imaging for incidental thyroid nodules (inverse) | For an incidental thyroid nodule under 1.0 cm on chest or neck CT or MRI (patients 18 or older), no follow-up imaging recommended |

**Not included:** measure 145 (fluoroscopy radiation exposure indices) needs every fluoroscopy procedure report, which many sites do not send to Guardian. Measure 494 is an electronic measure built on CT dose and image-quality data rather than report text.

Guardian also scores report-quality measures of its own, which are not MIPS measures, and you can write a measure as a plain clinical question with its procedure and diagnosis criteria.

## Results and export

The Quality tab shows each measure's performance rate, and for each exam the answer and the report words it rests on. For each measure and performance period, users who can edit quality measures (site admins, Quality Champions, and any role you give that permission) can download:

- **Case file (CSV):** one row per scored exam with its result and CMS quality-data code, date of service, accession, procedure and diagnosis codes, patient age, reporting radiologist and the quoted evidence.
- **Summary (CMS QPP format):** performance met, performance not met and the eligible population for the period.

Guardian is not a qualified registry or QCDR and does not submit to CMS. Your registry adds the clinician identifiers (NPI and TIN) and submits.

**Plan the period.** A measure scores new reports from the day it is turned on, plus recent reports inside your site's look-back window. Turn measures on at the start of the performance period. Guardian keeps measure results, but once a report's text is removed under your [data retention](security/data-retention.md) settings, that exam can no longer be scored again.

Quality measures are available in the standard app and the iframe when your plan includes them; see [Feature Support](feature-support.md).
