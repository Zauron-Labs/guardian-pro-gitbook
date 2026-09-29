# SaaS Onboarding

Guardian Pro SaaS is hosted and operated by Zauron. You don't install or run servers: Zauron creates your organization on the Guardian platform, connects it to your PACS and reporting systems, and keeps it updated. This page explains how onboarding works and what your team provides.

If your organization needs Guardian to run inside its own cloud or data center instead, see [Dedicated Installation](installation.md).

## What you get

- **Your own Guardian site** at a web address Zauron gives you, for the dashboard, viewer and [DataForge](dataforge/overview.md).
- **Your own data store.** Each customer's studies, reports, users and results are kept in a separate database and storage area. Sign-in to one customer's site never gives access to another's.
- **Your own integration settings.** Zauron configures your PACS, reporting and directory connections. Your administrators can see these settings in Guardian but don't need to maintain them.
- **Updates and monitoring** by Zauron, including the AI language model used to read reports and any AI imaging models you license.

The platform is hosted in Amazon Web Services (AWS) in the United States.

## Onboarding steps

| Step | What happens | Who |
|------|--------------|-----|
| 1. Intake | Complete the [SaaS onboarding checklist](saas-onboarding-checklist.csv): organization details, time zone, first administrators, sites, PACS, reporting and sign-in method. | You, with Zauron |
| 2. Site created | Zauron creates your Guardian site and sends your web address. Your first administrators can sign in. | Zauron |
| 3. Connectivity | Your network team sets up the site-to-site VPN (see [Connecting your network](#connecting-your-network)). Nothing flows between your network and Guardian until this step is complete. | You and Zauron |
| 4. PACS and reports | Zauron adds your PACS and report source, and both sides test them: a DICOM echo and test study retrieval, then a test report. | You and Zauron |
| 5. Sign-in | Connect your identity provider (SSO) or directory, or use email sign-in links. | You and Zauron |
| 6. AI models | Zauron enables the AI models you license and maps your procedure descriptions to them. | Zauron |
| 7. Access controls | Zauron applies your web access ranges and any sites allowed to embed Guardian. | You and Zauron |
| 8. Go-live | Test studies and reports flow end to end, and your administrators add users and confirm the [peer review options](options.md). | You and Zauron |

## Connecting your network

Guardian connects to the systems inside your network (PACS, reporting system and, if used, your LDAP directory) through a **site-to-site IPsec VPN** between your firewall and Zauron's VPN gateway. This is the standard connection for every SaaS customer.

### How the tunnel works

- Zauron assigns your organization a **dedicated public IP address**. Guardian's side of your tunnel is that single address. Your PACS, reporting system and directory talk to Guardian at it.
- You tell Zauron your **VPN gateway's public IP address** and the **internal addresses of the systems Guardian must reach**. Together these define the tunnel's traffic.
- Zauron's gateway starts the tunnel and restarts it if it drops. Your gateway must accept the connection.
- Traffic inside the tunnel is encrypted by IPsec, so DICOM and HL7 inside the tunnel don't need their own TLS.

### Tunnel settings

These settings are the same for every customer.

| Setting | Value |
|---------|-------|
| IKE version | IKEv2 |
| Authentication | Pre-shared key, at least 20 characters. Exchange it with Zauron over a secure channel, never by email. |
| IKE (phase 1) | AES-256, SHA-256, Diffie-Hellman group 14 (2048-bit MODP) |
| IPsec / ESP (phase 2) | AES-256-GCM (16-byte ICV), or AES-256 with SHA-256. Diffie-Hellman group 14 for perfect forward secrecy. |
| IKE lifetime | 8 hours |
| IPsec lifetime | 1 hour |
| Dead peer detection | 30 seconds |
| NAT traversal | Always on (IPsec is carried in UDP 4500) |

### Firewall rules on your side

| Direction | Between | Ports |
|-----------|---------|-------|
| Internet, both ways | Your VPN gateway ↔ Zauron's VPN gateway | UDP 500 and UDP 4500 |
| Inside the tunnel, from your network to Guardian | Your PACS → your Guardian address | The DICOM port Zauron assigns you |
| Inside the tunnel, from your network to Guardian | Your interface engine → your Guardian address | The HL7 port Zauron assigns you (HL7 only) |
| Inside the tunnel, from Guardian to your network | Your Guardian address → your PACS | Your PACS DICOM port |
| Inside the tunnel, from Guardian to your network | Your Guardian address → your reporting system | Its HTTPS or database port |
| Inside the tunnel, from Guardian to your network | Your Guardian address → your directory | LDAPS 636, or LDAP 389 with STARTTLS |
| Internet, to Guardian | Your users → your Guardian web address | HTTPS 443 |

### Connecting without a VPN (Direct)

If a VPN isn't possible, an **HL7 report feed** can connect directly over the internet instead. The connection is protected by:

- **TLS 1.2 or newer.** Guardian presents a publicly trusted certificate for your Guardian address.
- **A required client certificate.** Your interface engine must present a certificate signed by a CA you give Zauron (the CA certificate only, never a private key).
- **Source addresses.** Connections are accepted only from the public IP ranges you register.

Direct connections are available for HL7 only. PACS, PowerScribe and LDAP connections use the VPN. See the [HL7 Report Interface](integrations/hl7.md) for details.

## PACS

Guardian retrieves the studies it needs from your PACS:

- **Pull (standard):** Guardian queries your PACS by accession number (C-FIND) and asks it to send the study (C-MOVE) to Guardian.
- **Push:** your PACS or DICOM router sends studies to Guardian (C-STORE).

**You provide:**
- your PACS AE title, IP address and DICOM port;
- a separate PACS entry for each site whose images come from a different PACS.

**Zauron provides:**
- Guardian's AE title (unique to your organization);
- your Guardian IP address;
- the DICOM port.

Add Guardian to your PACS as a DICOM node with those values, and allow it to query and move studies. Guardian refuses studies sent to any other AE title.

## Reporting systems

Guardian reads finalized radiology reports from one of these sources per site:

| Source | How it connects |
|--------|-----------------|
| **PowerScribe** | Guardian signs in to the PowerScribe web API with a service account you create. |
| **HL7 v2 (ORU^R01)** | Your interface engine sends finalized reports to Guardian over MLLP. See the [HL7 Report Interface](integrations/hl7.md). |
| **PowerScribe database (SQL Server)** | Guardian reads finalized reports from the PowerScribe database with a read-only account, when the web API isn't used. |

Several sites can share one report source.

## Sign-in and users

Guardian has no shared passwords. Every person signs in as themselves, with one of:

- **Single sign-on (OIDC)**, recommended: Microsoft Entra ID, Okta, Ping or another OpenID Connect provider. You register Guardian as an application and give Zauron the issuer, client ID and client secret.
- **LDAP / Active Directory**, over the VPN only: LDAPS or STARTTLS, a bind account, the search base and your directory's CA certificate.
- **Email sign-in links**: no directory. Administrators add users in Guardian, and users sign in with a one-time link sent to their work email.

With SSO or LDAP, roles can follow your directory groups, or your administrators can set them in Guardian. The roles are **Admin**, **Champion**, **Radiologist** and **Trainee** (see [Training & Onboarding](training.md)). Your first administrators are named at intake. They add everyone else.

Radiologists receive their review assignments by email. Ask Zauron for the sending address so you can allow it in your mail filters.

## Web access and embedding

- **Web access ranges:** the public IP ranges your users connect from. Zauron restricts your Guardian address to these ranges. We strongly recommend setting them, even though users must sign in either way. The no-login Flag Case button from your PACS works only when ranges are set.
- **Embedding:** if you show Guardian inside another application in an iframe (for example a PACS or RIS web page), give Zauron those sites' addresses. Only the sites you list can embed your Guardian pages.

## Data protection

| Area | How Guardian SaaS protects it |
|------|-------------------------------|
| Separation between customers | A separate database, storage area and web address for each customer. Sign-in is bound to your own site. |
| Data in transit | HTTPS with TLS 1.2 or newer for users. IPsec VPN, or mutual TLS for Direct HL7, for your systems. TLS to the database. |
| Data at rest | Encrypted databases, disks and storage. Connection secrets, such as your PowerScribe password, are stored encrypted. |
| Images | DICOM private tags are removed from the copies Guardian keeps. Study identifiers and accession numbers are kept so your team can find the study in your PACS. |
| Logs | Patient medical record numbers are never written to logs. |
| Report text | Read by an AI language model hosted in AWS in the United States, reached over a private network connection rather than the public internet. |
| Audit | Sign-ins, study views, exports, reviews and administrative changes are recorded in your site's audit log. When Zauron support staff need to act in your site, access is time-limited and recorded in your audit log. |

For security documentation, contact [security@zauronlabs.com](mailto:security@zauronlabs.com).

## Go-live checklist

1. The VPN tunnel is up, and a DICOM echo works in both directions.
2. A test study is retrieved from your PACS and opens in the Guardian viewer.
3. A test report arrives from your reporting system and is linked to its study.
4. Your administrators and a test radiologist can sign in.
5. A test assignment email arrives, and its link opens the case.
6. Web access ranges are set, and any embedding sites are listed.
7. The [peer review options](options.md) are confirmed.

## Questions

Contact your Zauron onboarding lead, or [service@zauronlabs.com](mailto:service@zauronlabs.com) for technical support.
