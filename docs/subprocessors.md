# Subprocessors

A subprocessor is a third party that Zauron Labs uses to store or process customer data on your behalf. This page lists them for each way Guardian Pro is delivered.

*Last updated: October 24, 2026*

## Guardian Pro Cloud

Guardian Pro Cloud is hosted in Amazon Web Services (AWS) in the United States. Each customer has its own database, storage area and web address.

### Subprocessors that handle customer data

| Subprocessor | Service | Purpose | Customer data | Location | Agreement |
|---|---|---|---|---|---|
| Amazon Web Services, Inc. | Compute, database, storage, backups, encryption keys, monitoring | Hosting the platform, storing studies and reports, running AI image models | Reports, images and review data are stored and processed | United States | AWS Business Associate Addendum |
| Amazon Web Services, Inc. | Amazon Bedrock | AI language model that reads report text | Report text in each request. Bedrock does not store it or use it to train models. | AWS regions in the United States | AWS Business Associate Addendum |
| Microsoft Corporation | Azure Communication Services (Email) | Review assignments, notifications and sign-in links | Recipient name and email address, and the notification text. Accession numbers are off by default. Emails never contain report text or images. | United States (the data location of our email service) | Microsoft HIPAA Business Associate Agreement, included in the Microsoft Data Protection Addendum |
| Microsoft Corporation | Microsoft Azure (container hosting) | Running some AI image models in containers that Zauron operates | Images of the studies those models read | United States (East US 2) | Microsoft HIPAA Business Associate Agreement, included in the Microsoft Data Protection Addendum |

Azure container hosting is not used when **Restrict outside vendors** is on (see below).

### Optional subprocessors

These are used only when switched on for your site, and never when **Restrict outside vendors** is on.

| Subprocessor | Purpose | Customer data |
|---|---|---|
| Datadog, Inc. | Application monitoring, when your organization asks for it | Application logs for your site |
| AI model vendors | AI image models hosted by their own vendor, when you license one | Images of the studies the model reads |

### Restrict outside vendors

Each Cloud customer has a **Restrict outside vendors** setting. It is **on by default for new Cloud customers**. While it is on:

- AI image models run only inside the Guardian Pro Cloud platform in AWS. Requests to any other model host are refused.
- The AI language model is Amazon Bedrock only.
- Datadog is not used for your site.

With the setting on, the only subprocessors that handle your data are AWS (hosting and Amazon Bedrock) and Microsoft (email).

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
