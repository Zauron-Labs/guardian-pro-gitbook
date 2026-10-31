# Frequently Asked Questions

## Radiologists

### The system assigned me an exam type I don't regularly interpret. What should I do?

Guardian Pro includes protections to ensure you only receive cases matching your practice profile. If a mismatch occurs, you have several options:

- Click **Defer** on the case to remove it and prevent similar future assignments.
- Contact your admin to adjust the thresholds that define "regularly interpreted" exams.
- Request your admin exclude you from specific modalities, body parts, CPT codes, or combinations thereof.

### My case list is empty. Is that expected?

No. The system should always assign a minimum number of cases. An empty list indicates a system issue — please alert your administrator, who can work with the Zauron team to resolve it.

### How do I review the images and studies I've been assigned?

Your email contains a **Launch Peer Review** button. Clicking it starts your case list. After completing each case, you are automatically presented with the next one. A progress indicator shows how many cases remain. If you need to stop at any point, simply close the window — the system will resume where you left off the next time you launch.

### Are all reviews double-checked by an adjudicator?

Only reviews that meet or exceed your institution's configured threshold (e.g., a score of 2b or higher) are automatically escalated to one or more adjudicators. Optionally, the original interpreting radiologist is also notified.

### I disagree with a Learning Opportunity I received. What can I do?

Learning Opportunities are informational only and do not require acknowledgment. The criteria your organization uses to identify Learning Opportunities can be adjusted in coordination with the Zauron team.

### I'm going on parental or other extended leave. Will cases accumulate while I'm away?

Cases assigned during your absence will automatically age out after your organization's pre-defined window (default: 30 days). No action is required on your part.

### I'm a combined Breast/Body attending. Do I have to participate in peer review?

Medicare requires each radiology department to maintain accreditation, which typically includes regular peer review of each radiologist — though not necessarily active participation as a reviewer. We recommend full participation but are happy to coordinate with your department's existing peer review strategy. Additionally, Zauron Labs offers a separate service leveraging our Radiology Network to perform external peer reviews, filling any gaps in your program and helping maintain compliance.

### Can we leave a radiologist's reports out of peer review?

Yes. An administrator turns off **Peer reviewed** for that person in **Admin → Users**. Guardian then never chooses their reports for peer review, including external peer review by Zauron radiologists. Cases already assigned stay assigned, and any of their cases can still be flagged by hand with Flag Case. Turn it back on at any time; their new reports are eligible again from the next assignment run.

## Integrations

### Can we use our own viewer or worklist instead of Guardian's screens?

Yes. With the [Guardian API](guardian-api.md), your viewer or worklist can read exams and AI findings and complete peer reviews, while Guardian keeps choosing and assigning the cases. Ask Zauron to switch the API on for your site; your administrators then create API keys.

## Images and retention

### How long does Guardian keep study images, and what happens if a study is gone when someone needs it?

Your PACS stays the system of record. Guardian keeps its own viewer copy of a study for as long as your organization's data retention settings allow, then removes it. Studies that are in use are kept longer:

- **Peer review assignments** stay available for your review window (30 days by default) after they are assigned.
- **Flag Case and other reviews outside an assignment** stay available for the same window after the last activity on the review: when the case was flagged, submitted, finalized or last worked on by a reviewer.
- **Reviewed studies** with a determination your site keeps, a Great Call or a technical issue are kept for the longer period in your retention settings.
- **Peer Learning PowerPoint** cases stay available while the slide links work (up to 28 days).

When a review or a PowerPoint export needs a study whose images are no longer on Guardian, or were never copied to Guardian, Guardian retrieves it from your PACS again automatically. The review or slide link shows the images once that retrieve finishes.
