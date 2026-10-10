# Security

Guardian Pro is built with security at its core, meeting rigorous standards required by healthcare organizations and government agencies.

## Certifications

### Texas Risk and Authorization Management Program (TX-RAMP)

Guardian Pro is certified under the [Texas Risk and Authorization Management Program (TX-RAMP)](https://dir.texas.gov/information-security/texas-risk-and-authorization-management-program-tx-ramp). TX-RAMP provides a standardized approach for security assessment, certification, and continuous monitoring of cloud computing services that process the data of Texas state agencies.

This certification demonstrates that Guardian Pro meets the stringent security requirements established by the Texas Department of Information Resources (DIR) for:

- **Security Assessment**: Comprehensive evaluation of security controls and practices
- **Authorization**: Formal approval to process sensitive state agency data
- **Continuous Monitoring**: Ongoing security oversight and compliance verification

## Access control

- **Individual accounts only.** Every person signs in as themselves, through single sign-on (OIDC), LDAP / Active Directory, or a one-time link sent to their work email. There are no shared or default passwords, including for administrators.
- **Roles.** Admin, Champion, Radiologist and Trainee, assigned in Guardian or taken from your directory groups. Access is re-checked as people use Guardian, so a removed user or changed role takes effect promptly.
- **Assignment links** that radiologists receive by email open only the assigned cases and expire.
- **Network restrictions.** Access can be limited to your organization's network ranges. The no-login Flag Case button from your PACS works only from those ranges and is rate-limited.
- **Embedding.** Only sites you list can show Guardian inside their pages.
- **Guardian API keys.** The REST API for your own systems is off until Zauron switches it on for your site. Each key is scoped, limited to the IP ranges you list, expires, and can be revoked at once; Guardian stores only a hash of it. Patient data is returned only to keys with that scope, and each access is audited. See the [Guardian API](guardian-api.md).
- **Zauron support access** to your site is time-limited and recorded in your audit log.

## Data protection

- **In transit:** TLS 1.2 or newer for all web traffic. SaaS integrations with your network use an IPsec site-to-site VPN, or mutual TLS for direct HL7 feeds. Database connections use TLS.
- **At rest:** databases, disks and storage are encrypted. Integration secrets such as service-account passwords are stored encrypted.
- **Images:** DICOM private tags are removed from the copies Guardian keeps. Study identifiers and accession numbers are kept so your team can locate the original study.
- **Logs:** patient medical record numbers are never written to logs.
- **Customer separation (SaaS):** each customer has its own database, storage area and web address. See [SaaS Onboarding](saas-onboarding.md#data-protection).

## Audit

Each site keeps an append-only audit log of sign-ins and failed sign-ins, study views, exports, AI processing, review submissions, Guardian API key changes and patient-data access through the API, and administrative changes. Each entry records who acted, their role and the source address.

## Questions

For security-related inquiries or to request our security documentation, contact our team at [security@zauronlabs.com](mailto:security@zauronlabs.com).
