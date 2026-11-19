# Security

Guardian Pro is built with security at its core, meeting rigorous standards required by healthcare organizations and government agencies.

This page is the overview. The details for security reviewers are on these pages:

- [Encryption in transit](security/encryption-in-transit.md): web, database, DICOM and HL7 connections
- [Audit log](security/audit-log.md): what is recorded, the record format, export and SIEM streaming
- [Data retention](security/data-retention.md): how long each kind of data is kept, and the settings
- [Container security](security/container-security.md): how the containers are locked down, and the documented exceptions
- [Verifying images](security/image-verification.md): signed release images, digest pins, and checking them yourself
- [Secrets in your cloud secret store](security/secrets.md): AWS Secrets Manager, AWS Systems Manager Parameter Store, Azure Key Vault
- [Outbound connections](security/outbound-connections.md): everything a dedicated installation sends outside your network
- [Subprocessors](subprocessors.md): the third parties that store or process customer data

## Certifications

### Texas Risk and Authorization Management Program (TX-RAMP)

Guardian Pro is certified under the [Texas Risk and Authorization Management Program (TX-RAMP)](https://dir.texas.gov/information-security/texas-risk-and-authorization-management-program-tx-ramp). TX-RAMP provides a standardized approach for security assessment, certification, and continuous monitoring of cloud computing services that process the data of Texas state agencies.

This certification demonstrates that Guardian Pro meets the stringent security requirements established by the Texas Department of Information Resources (DIR) for:

- **Security Assessment**: Comprehensive evaluation of security controls and practices
- **Authorization**: Formal approval to process sensitive state agency data
- **Continuous Monitoring**: Ongoing security oversight and compliance verification

## Access control

- **Individual accounts only.** Every person signs in as themselves, through single sign-on with your OpenID Connect (OIDC) identity provider, or with a one-time link sent to their work email. There are no shared or default passwords, including for administrators. When single sign-on is on for your organization, its users sign in only through it, and signing out of Guardian also signs them out of the identity provider.
- **Directory sync.** Guardian can read users and roles from LDAP / Active Directory. The directory is the source of names, email addresses and roles; Guardian never checks directory passwords. Directory users sign in with single sign-on or a one-time email link.
- **Roles.** Admin, Champion, Radiologist and Trainee, assigned in Guardian or taken from your directory groups. Access is re-checked as people use Guardian, so a removed user or changed role takes effect promptly.
- **Sign-out after inactivity**, checked by the server: see [below](#sign-out-after-inactivity).
- **Admin allowed networks.** You can limit administration to the IP ranges you name. From any other address, the Admin tab, admin settings, admin through an embedding site, the viewer's admin settings, DataForge administration (including DataForge admin API keys) and Zauron support sessions are refused, and each refusal is recorded in your audit log. Other users are not affected. This list is narrower than your web access ranges, which cover the whole site. Ask Zauron to set it; it is blank (any network) by default.
- **Assignment links** that radiologists receive by email open only the assigned cases and expire. The **Flag a Case** link in those emails signs the recipient in as themselves (never as an administrator); it expires a few hours after the email is sent and stops working when they sign out.
- **Network restrictions.** Access can be limited to your organization's network ranges.
- **Flag Case button from your PACS.** On Guardian Pro Cloud and on installations Zauron hosts, the button requires a sign-in. Only a dedicated installation your organization runs can let the button open a case without a sign-in; it then works only from your network ranges, opens only the case clicked, and is rate-limited.
- **Embedding.** Only sites you list can show Guardian inside their pages.
- **Guardian API keys.** The REST API for your own systems is off until Zauron switches it on for your site. Each key is scoped, limited to the IP ranges you list, expires, and can be revoked at once; Guardian stores only a hash of it. Patient data is returned only to keys with that scope, and each access is audited. See the [Guardian API](guardian-api.md).
- **Zauron support access** to your site is time-limited and recorded in your audit log, including every read and change made during it.

### Sign-out after inactivity

The server ends idle sessions, so a forgotten or copied browser tab cannot keep a session alive. Only keyboard and mouse input counts; background refreshes do not.

| Session | Signed out after | Notes |
|---|---|---|
| Administrator (Admin tab, admin settings, admin through an embedding site) | 30 minutes without input | Adjustable from 5 minutes to 12 hours, or off |
| Every dashboard user | Off by default | Can be set for your site; the shorter limit applies to administrators |
| DataForge | 60 minutes without input | Adjustable from 5 minutes to 12 hours. Every DataForge session also ends 12 hours after sign-in (by default) |
| Zauron support session in your site | 60 minutes without input | Ends 12 hours after it starts at the latest |
| Zauron's operator console | 30 minutes without input | Used by Zauron staff only |

## Data protection

- **In transit:** TLS 1.2 or newer for all web traffic. Guardian Pro Cloud connects to your network over an IPsec site-to-site VPN. Database connections use TLS with the server certificate verified. DICOM and HL7 can also run over TLS. See [Encryption in transit](security/encryption-in-transit.md).
- **At rest:** databases, disks and storage are encrypted. Integration secrets such as service-account passwords are stored encrypted. A dedicated installation can keep its secrets in your own cloud secret store; see [Secrets in your cloud secret store](security/secrets.md).
- **Images:** DICOM private tags are removed from the copies Guardian keeps. Study identifiers and accession numbers are kept so your team can locate the original study.
- **Logs:** patient medical record numbers are never written to logs.
- **Web pages:** the Guardian dashboard loads no scripts or fonts from third-party sites; everything it needs is served by your Guardian site.
- **Customer separation (Cloud):** each customer has its own database, storage area and web address. See [Cloud Onboarding](cloud-onboarding.md#data-protection).
- **Retention:** report text, review records and activity logs can be deleted on a schedule you choose. See [Data retention](security/data-retention.md).
- **Subprocessors:** the third parties that store or process customer data are listed on [Subprocessors](subprocessors.md).

## Software supply chain

- **Signed images.** Guardian Pro release images are signed, and each release tag is locked in Zauron's registry so it always names the same image. See [Verifying images](security/image-verification.md).
- **Pinned by digest.** Upgrades pin each image by its content digest, and can be set to refuse any image whose signature does not verify.
- **Hardened containers.** Services run as a non-root user with all Linux capabilities dropped. See [Container security](security/container-security.md).

## Audit

Each site keeps an append-only audit log, kept for at least six years and never purged. It records sign-ins, failed sign-ins and sign-outs, study views, patient-data reads and exports, every administrative change, every request refused for missing sign-in or permission, Guardian API key changes and patient-data access through the API, and Zauron support activity. Each entry records who acted, their role, the source address and whether the action succeeded, failed or was denied. Administrators can export the log, and a dedicated installation can stream it to your SIEM. See [Audit log](security/audit-log.md).

## Questions

For security-related inquiries or to request our security documentation, contact our team at [service@zauronlabs.com](mailto:service@zauronlabs.com).
