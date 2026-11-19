# Outbound connections

Every connection that a **dedicated installation your organization runs** makes to something outside your environment: what it carries, whether it is on by default, and how to turn it off. Connections to your own systems (PACS, reporting system, interface engine, identity provider, directory, your cloud secret store and storage) are not listed.

For Guardian Pro Cloud, see [Subprocessors](../subprocessors.md). The **Restrict outside vendors** setting described there is a Cloud setting; on an installation you run, you choose the vendors, and the switches below are the controls.

## Summary

| Connection | Destination | What it carries | Default | How to turn it off |
|---|---|---|---|---|
| Licence check | Zauron's container registry | The installation's registry identity, its address and the image names checked. No patient data, no usage counts | On, every 15 minutes | Zauron can switch technical licence checks off for your installation (the licence then rests on the agreement alone) |
| Image pulls and update checks | Zauron's container registry | Registry credentials and image or version queries. No patient data | On: model catalog daily, registry health every 30 minutes, image pulls on upgrades and when a model starts | Mirror the images into your own registry ([signatures included](image-verification.md#mirroring-into-your-own-registry)) |
| AI language model | AWS: Amazon Bedrock in your account. Azure, Google, Self-hosted: a Llama 4 Scout endpoint Zauron runs in Microsoft Foundry for your installation only, unless you provide your own | Report text (see below) | On (Guardian needs it to read reports) | Use an endpoint in your own cloud account (Amazon Bedrock or Azure) |
| Email | Your SMTP relay; or Azure Communication Services when Zauron provides email | Every Guardian email: sign-in links, review links, review comments and counts. Accession numbers only when you switch them on; never report text or images | Your relay | Use your own relay |
| AI image models | Model containers on the Guardian host; an outside model endpoint only for a model you approve | De-identified images (see below) | Local containers. New models start inactive | Per model: inactive, or run as a local container |
| System health alerts | The recipients your administrators set | Alert names and messages, environment name, check time and a link to your dashboard. No patient data | No Zauron recipient on an installation you run | Admin → System alerts |
| TLS certificates (Let's Encrypt) | Let's Encrypt; public DNS resolvers only for the DNS challenge | Your Guardian host names, and a Zauron contact address for the certificate account | Off (your certificate, or self-signed for a lab) | Use your own certificate |
| Datadog | Your organization's own Datadog account | Allow-listed log and metric fields (route, status, latency, counts), with patient data removed | Off | Leave it off |
| DataForge GPU service (RunPod) | RunPod | For AI-assisted labeling in DataForge: an image slice and the labeler's prompts, or an image volume | Off; needs a key you provide | Leave it off |
| eRadPeer (ACR) | The ACR eRadPeer service | Your organization's eRadPeer credentials, to test the connection | Off | Leave it off |

## Details

### Report text and the AI language model

Before report text is sent, Guardian blanks the exam's accession and order numbers and labelled patient identifiers (MRN, date of birth, name, SSN, phone number, email address). Names and dates that appear without a label stay in the text. The endpoint also receives a short health-check request every five minutes.

When the language model endpoint is the one Zauron runs, report text leaves your environment to Microsoft under Zauron's agreement with Microsoft (see [Subprocessors](../subprocessors.md#dedicated-installation)); your agreement with Zauron covers it. With Amazon Bedrock or an Azure endpoint in your own account, report text stays in your account.

### Images sent to an outside AI model

Guardian starts licensed AI models as containers on the Guardian host, and new models start inactive. A model at an outside HTTPS address (a vendor's, or one Zauron hosts in Azure) is used only after it is approved for your installation. Images sent to a model keep their pixels, with headers de-identified: study identifiers replaced, private tags removed, free text blanked. Scanned documents, objects with burned-in text and encapsulated documents are not sent.

### What Guardian does not do

- No telemetry, analytics, crash or error reporting, or "phone home" update checks. The web entry point's version check and anonymous usage reporting are off.
- The Guardian dashboard loads no scripts or fonts from third-party sites.
- The report-search model runs offline on the Guardian host; report text sent to it never leaves the host.
- Licensing is checked only through the registry (above); there is no separate licence server.
- Zauron support sessions are inbound: the operator's browser opens your Guardian site with a one-time ticket. Your installation makes no outbound call for them.
- The Zauron reader pool exists only on Guardian Pro Cloud; an installation you run has no connection to it.

## Questions

Contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
