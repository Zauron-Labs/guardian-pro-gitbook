# Subprocessors

A subprocessor is a third party that Zauron Labs uses to store or process customer data on your behalf. This page lists them for each way Guardian Pro is delivered.

*Last updated: October 24, 2026*

## Guardian Pro Cloud

Each Guardian Pro Cloud instance is hosted in either Amazon Web Services (AWS) or Microsoft Azure, in the United States. The AI image model containers for an instance also run in AWS or Azure, chosen for that instance, and this can be a different cloud from the one that hosts the instance. Your agreement states which cloud is used for each. Each customer has its own database, storage area and web address.

### Subprocessors that handle customer data

| Subprocessor | Service | Used for | Purpose | Customer data | Location | Agreement |
|---|---|---|---|---|---|---|
| Amazon Web Services, Inc. | Compute, database, storage, backups, encryption keys, monitoring | Instances hosted in AWS | Hosting the platform, storing studies and reports | Reports, images and review data are stored and processed | United States | AWS Business Associate Addendum |
| Amazon Web Services, Inc. | Amazon Bedrock | Instances hosted in AWS | AI language model that reads report text | Report text in each request. Bedrock does not store it or use it to train models. | AWS regions in the United States | AWS Business Associate Addendum |
| Microsoft Corporation | Microsoft Azure (compute, database, storage, backups, encryption keys, monitoring) | Instances hosted in Azure | Hosting the platform, storing studies and reports | Reports, images and review data are stored and processed | United States | Microsoft HIPAA Business Associate Agreement, included in the Microsoft Data Protection Addendum |
| Microsoft Corporation | Azure OpenAI in Azure AI Foundry | Instances hosted in Azure | AI language model that reads report text | Report text in each request. Microsoft does not use it to train models. | Azure regions in the United States | Microsoft HIPAA Business Associate Agreement, included in the Microsoft Data Protection Addendum |
| Amazon Web Services, Inc. or Microsoft Corporation | AI image model containers that Zauron operates | Every instance, in the cloud chosen for it | Running AI image models | Images of the studies those models read | United States | AWS Business Associate Addendum or Microsoft HIPAA Business Associate Agreement |
| Microsoft Corporation | Azure Communication Services (Email) | Every instance | Review assignments, notifications and sign-in links | Recipient name and email address, and the notification text. Accession numbers are off by default. Emails never contain report text or images. | United States (the data location of our email service) | Microsoft HIPAA Business Associate Agreement, included in the Microsoft Data Protection Addendum |

### Optional subprocessors

These are used only when switched on for your site, and never when **Restrict outside vendors** is on.

| Subprocessor | Purpose | Customer data |
|---|---|---|
| Datadog, Inc. | Application monitoring, when your organization asks for it | Application logs for your site |
| AI model vendors | AI image models hosted by their own vendor, when you license one | Images of the studies the model reads |

### Restrict outside vendors

Each Cloud customer has a **Restrict outside vendors** setting. It is **on by default for new Cloud customers**. While it is on:

- AI image models run only in the containers Zauron operates in the cloud chosen for your instance. Requests to any other model host are refused.
- The AI language model is the one in your instance's hosting cloud: Amazon Bedrock in AWS, or Azure OpenAI in Azure.
- Datadog is not used for your site.

With the setting on, the only subprocessors that handle your data are AWS, Microsoft, or both, depending on where your instance and its AI image models run. Microsoft always provides email.

## Dedicated installation

A dedicated installation runs in your organization's own cloud account or data center. Its hosting, AI language model, AI model hosts and email service are your own, so Zauron adds no subprocessors to it. Zauron provides the software as container images, and Zauron support access to your installation is time-limited and recorded in your audit log.

If Zauron hosts a dedicated installation for you, its subprocessors are listed in your agreement.

## Vendors without access to customer data

| Vendor | Purpose |
|---|---|
| Microsoft Azure (container registry) | Distributing Guardian Pro software images |
| GitHub, Inc. | Source code hosting |
| Let's Encrypt (ISRG) | TLS certificates |
| jsDelivr, Google Fonts | Static files (a text-editor library and fonts) that your browser downloads |

## Changes to this list

When we add a subprocessor that handles customer data, we update this page and notify customer administrators by email at least 30 days in advance.

## Questions

For a copy of a business associate agreement or other questions about subprocessors, contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
