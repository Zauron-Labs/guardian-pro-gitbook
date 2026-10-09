# Peer Learning Decisions

Before go-live, your quality leadership answers a short set of policy questions: what counts as peer learning, how much each radiologist does each week, how AI is used, who hears about errors, and how cases feed conferences and the teaching file. Each question below maps to a setting your site administrator can change in the Guardian dashboard.

For each question, Zauron recommends a **Zauron Gold Standard**. Accept it as-is or write in your own policy.

## Decision Worksheet

Download the [Peer Learning Decisions Worksheet (Excel)](peer-learning-decisions.xlsx). It has a Yes/No drop-down for each decision and a second tab showing where each setting lives in the dashboard. A plain [CSV version](peer-learning-decisions.csv) is also available.

1. **Review** each decision and the Zauron Gold Standard.
2. **Choose** Yes or No in the "Use Zauron Gold Standard" column. Where you choose No, describe your site's policy when you return the worksheet.
3. **Return** the completed worksheet to your Zauron representative, or have your site administrator apply it in the dashboard (see [Where to set it](#where-to-set-it)).

### The Zauron Gold Standard at a glance

1. Viewing interesting cases counts as peer learning activity.
2. Flagging a case as a Great Call or for quality review counts as peer learning activity.
3. Target 5 cases per week of activity.
4. Use Guardian's outputs to drive peer learning conferences (M&M, topic-driven conferences).
5. Original authors are notified of potential and confirmed errors.

## What counts as peer learning activity

Peer learning activity counts toward each radiologist's "progress to date" in the weekly email and toward the site's Quality Improvement Activity in Compliance.

| Decision | Zauron Gold Standard | Use Zauron Gold Standard (Yes/No) | Goal |
|----------|----------------------|-----------------------------------|------|
| Does completing assigned peer review count as peer learning activity? | Yes. Every completed assigned review counts. | | Credit the core review work in each radiologist's progress and in the site's compliance record. |
| Does viewing an interesting case count as peer learning activity? | Yes. Opening an interesting case counts (once per case). | | Credit case-based learning, not only scoring, so teaching cases count toward participation. |
| Does flagging a case count as peer learning activity? | Yes. Flagging a case as a Great Call or for quality review counts. | | Encourage radiologists to raise great calls and quality concerns as part of everyday reading. |
| Does feedback on interesting cases (thumbs up / down) count? | Yes. | | Learn which cases teach well and credit the time radiologists spend on them. |
| Does viewing a colleague's Great Call count? | Yes. Opening a Great Call from the weekly email counts (once per case). | | Spread good practice by crediting radiologists who learn from colleagues' great calls. |

## Case volume and random sampling

| Decision | Zauron Gold Standard | Use Zauron Gold Standard (Yes/No) | Goal |
|----------|----------------------|-----------------------------------|------|
| How many cases per week should each radiologist complete? | Target 5 cases per week of activity. | | Steady, sustainable participation that meets accreditation expectations without adding burden. |
| What is the most cases a radiologist can be assigned in a week? | 10 cases. | | Cap workload to prevent reviewer fatigue, even in weeks when AI finds many candidate cases. |
| Should random cases be included, or only AI-selected cases? | Yes. Include at least 1 random case in every weekly assignment. | | Keep an unbiased baseline sample alongside AI-selected cases for quality metrics and accreditation. |

## AI-driven mistake triage

| Decision | Zauron Gold Standard | Use Zauron Gold Standard (Yes/No) | Goal |
|----------|----------------------|-----------------------------------|------|
| Should AI drive mistake triage (choose likely discrepancies for review)? | Yes. Production AI models select likely discrepancies for review; new models run in test mode before they affect assignments. | | Spend reviewer time on the cases most likely to hold a learning opportunity. |
| How accurate should AI-selected cases be? | Zauron tunes each model's thresholds for an enhanced positive predictive value (PPV) of 0.5: about 1 in 2 AI-selected cases holds a true discrepancy. | | A consistent hit rate across models: enough true findings to be worth reviewing, few enough false alarms to keep radiologists' trust. |

## Feedback to original authors and escalation

| Decision | Zauron Gold Standard | Use Zauron Gold Standard (Yes/No) | Goal |
|----------|----------------------|-----------------------------------|------|
| Should original authors be told about potential errors (AI-detected, before peer review)? | Yes. Turn on the self-review pathway so authors privately receive AI-confirmed potential discrepancies on their own reports. | | Fast, private, non-punitive feedback that lets radiologists self-correct and addend. |
| Should original authors be told about confirmed errors (discrepant peer review)? | Yes. Notify the original author for every discrepant peer review response. | | Close the loop so every confirmed discrepancy becomes a learning moment for the author. |
| Should a Quality Champion be told about significant discrepancies? | Yes. Notify the original author and the domain champion for moderate (2b) and higher discrepancies. | | Make sure clinically significant discrepancies get timely follow-up from quality leadership. |
| Should original authors be told about their Great Calls? | Yes. Notify the original author of every Great Call. | | Positive reinforcement: recognize excellent reads, not only errors. |
| Who adjudicates flagged and disputed cases? | One or more Quality Champions covering each subspecialty, given the Champion role. | | A clear, qualified owner for every escalation and discrepancy decision. |

## Conferences and the teaching file

| Decision | Zauron Gold Standard | Use Zauron Gold Standard (Yes/No) | Goal |
|----------|----------------------|-----------------------------------|------|
| Should system outputs drive peer learning conferences? | Yes. Build M&M and topic-driven conferences from Guardian Search (M&M / Discrepancy, Critical Findings, MIPS / Quality) and export the case basket to PowerPoint. | | Turn individual reviews into group learning built from your own local cases. |
| Which peer review scores count as discrepancies (and feed M&M)? | Use the RADPEER-style scale as shipped: minor, moderate, major and complete-discordance scores count as discrepant. | | A consistent, recognized definition of discrepancy across reviewers, reports and conferences. |
| Should reviewers have to comment on discrepant scores? | Yes. Require a comment for every discrepant score. | | Comments make a case teachable at conference and actionable for the author. |
| What should the interesting cases (teaching file) emphasize? | Each weekly email shows 5 For You cases matched to the radiologist's own practice plus 5 top-scoring cases site-wide. | | A living teaching file: relevant to each radiologist's practice and showing the site's most instructive cases. |
| How many Great Calls should appear in the weekly email? | 5. | | Celebrate and share excellent reads across the group every week. |
| Should leadership receive a monthly compliance summary? | Yes. Turn on the automatic monthly email to administrators and Champions. | | Keep leadership informed of participation and quality trends without manual reporting. |

## Where to set it

Site administrators apply these decisions in the Guardian dashboard. Most are set per employer in **Admin → Policies → Employers → Edit**.

| Decision area | Dashboard location | Setting |
|---------------|--------------------|---------|
| What counts as peer learning activity | Admin → Policies → Employers → Edit → **Progress credit** | One checkbox per activity: peer review, manual flag case, interesting-case feedback, interesting case viewed, great call viewed |
| Cases per week | Admin → Policies → Employers → Edit → **General** | Week Assign Min (5), Week Assign Max (10) |
| Random cases | Admin → Policies → Employers → Edit → **General** | Random Exams Min (1) |
| AI-driven triage | Model-Zoo → Model Portfolio → edit model | Active, Test mode, Display threshold (Zauron tunes each model for an enhanced PPV of 0.5) |
| Potential errors to authors | Admin → Policies → Employers → Edit → **Self-review discrepancy emails** | Enable self-review pathway; then turn on **Self Discrepancy** for each radiologist in Admin → Users |
| Confirmed errors, champion alerts and Great Calls | Admin → Policies → Employers → Edit → **Notification Rules (Per Response)** | For each response type, choose the recipient (for example, "Original author of exam" or "Original author + domain champion") and whether to email immediately |
| Who adjudicates | Admin → Users and Admin → Roles | Assign the Champion role, which includes "Adjudicate flagged cases" |
| Discrepancy definition and required comments | Admin → Policies → **Peer Review Options** | "Count as peer-review discrepancy (M&M default filter)" and "Require comment in viewer" on each score |
| Conferences | Search → M&M / Discrepancy, Critical Findings, MIPS / Quality | Add cases to the basket and export to PowerPoint |
| Interesting cases and Great Calls in the weekly email | Admin → Notifications → **Picker emails** | Interesting cases: For You top N (5), Global top N (5). Great calls: maximum to show (5) |
| Monthly compliance summary | Admin → Notifications → **Alert routing** → Monthly Compliance Email | Enable automatic monthly send; choose recipients |

Interesting cases and Search depend on your Guardian Pro plan. If a setting above is not visible in your dashboard, contact your Zauron representative.

Operational settings such as case-volume floors for reviewer eligibility, trainee participation and AI governance are on the [Configuration Options](options.md) worksheet.

## Need Help?

- **Email**: [service@zauronlabs.com](mailto:service@zauronlabs.com)
- **Consultation**: Schedule a peer learning policy review with your Zauron implementation specialist.
