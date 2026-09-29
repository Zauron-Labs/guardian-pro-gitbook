# DataForge

DataForge is Guardian Pro's imaging data workspace. Teams use it to build labeled imaging datasets and to measure how AI models perform on their own patients. It runs inside your Guardian Pro deployment, next to the dashboard and viewer, so studies and labels stay in the same environment as the rest of your Guardian data.

Open it from the Guardian dashboard, or go to `https://<your Guardian address>/dataforge/`. You sign in with your Guardian account.

## Projects

Everything in DataForge belongs to a **project**. There are two kinds:

| Project type | Use it to |
|--------------|-----------|
| **Labeling** | Build a labeled dataset. Load studies, define the labels, assign studies to annotators and reviewers, and export the result. |
| **Model validation** | Measure an AI model against radiologist ground truth. Studies are grouped by the model's confidence, reviewers confirm or reject each result, and DataForge reports positive predictive value and agreement. |

Each project has its own members. A member is a **project admin**, an **annotator** or a **reviewer**. Project admins manage the project's settings, data, members and API keys.

## Labeling projects

A labeling project moves through these steps:

1. **Load studies.** Upload a DICOM archive or point DataForge at a folder on the server. Studies can be de-identified on the way in.
2. **Add reports** (optional). Import the radiology reports so annotators can read them next to the images.
3. **Define the label contract.** Choose the labels annotators may use. Each label is one of these kinds:
   - a box drawn on an image;
   - a classification of the whole study, one series or one image.
4. **Add AI prelabels** (optional). Import model predictions. Predictions that match the label contract become editable starting labels, so annotators correct them instead of drawing from scratch.
5. **Assign work.** Assign studies to annotators and reviewers. Assignees get an email with a link to their work.
6. **Annotate and review.** Annotators label each study. Reviewers agree or disagree with each label, send work back when changes are needed, and finalize the study. Finalized labels are locked.
7. **Follow progress.** The project shows how many tasks are assigned, in progress and complete, per person and per role.
8. **Export.** Download the results as a CSV, or as a packet containing the labels, the task history and overlay images. You can limit the export to labels that reviewers agreed with.

## Model validation and structured assessments

From the Guardian dashboard's **Model Zoo**, a site admin or quality champion can start a **structured assessment** of a licensed model. Guardian creates a dedicated DataForge project for it:

- The project holds the cohort of studies the model ran on, with the model's results and the matching radiology reports.
- Studies are sampled across the model's confidence range, so reviewers see both high-confidence and borderline results.
- The assessment is locked. Its cohort, ground truth and model results cannot be changed after launch, so the results can be relied on for governance.
- DataForge opens in a new tab as soon as the assessment is launched and shows progress while the studies are prepared.

Each assessment gets its own project, so a later assessment never changes the results of an earlier one.

## Analytics

Each project has a **Performance** view:

- **Model performance:** positive predictive value by confidence range, and a confusion matrix against the radiologist's reading.
- **Annotator quality:** how often reviewers agreed with each annotator's labels.
- **Adjudication:** study-level review decisions.

## Privacy

- Studies can be de-identified when they are loaded.
- A privacy officer can exclude individual series or images from a project. Excluded images are hidden from annotators and left out of exports and downloads.
- Every change is recorded with the person who made it.

## Working without the browser

Researchers and data teams can work with DataForge from their own code with the [DataForge Python SDK](sdk.md). It covers:

- loading images, reports and AI predictions;
- setting the label contract;
- assigning work and following progress;
- reading and importing labels;
- exporting results and downloading images.

Access is by API key:

| Key | Issued in | Can act on |
|-----|-----------|------------|
| **Project key** | The project's **Project Settings → API keys**, by a project admin | That one labeling project |
| **Admin key** | Guardian **Admin → Settings → DataForge module API keys**, by a site administrator | Every project of your site. Used to create and configure projects and to issue project keys. |

The SDK installs from your own Guardian deployment and does not need internet access. The full guide is also available inside DataForge under **Docs → SDK**.
