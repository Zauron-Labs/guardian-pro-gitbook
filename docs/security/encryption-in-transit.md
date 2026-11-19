# Encryption in transit

What Guardian Pro encrypts on the wire, what it can encrypt on request, and what stays unencrypted inside the Guardian host (and why).

## Summary

| Connection | Default | Available on request |
|---|---|---|
| Browsers and API clients → Guardian | HTTPS, TLS 1.2 or newer, ECDHE key exchange with AEAD ciphers, HSTS | — |
| Guardian → PostgreSQL database | TLS, server certificate and host name verified | — |
| Guardian → your LDAP / Active Directory | LDAPS or STARTTLS, directory certificate verified | — |
| Guardian Pro Cloud ↔ your network (PACS, reporting, directory) | IPsec site-to-site VPN | — |
| Your interface engine → Guardian HL7 listener (dedicated installation) | Plain MLLP inside your network | MLLP over TLS, or mutual TLS |
| Guardian → your PACS (C-ECHO, C-FIND, C-MOVE) (dedicated installation) | Plain DICOM inside your network | DICOM TLS per PACS, with an optional Guardian client certificate |
| Your PACS → Guardian DICOM receiver (C-STORE) (dedicated installation) | Plain DICOM inside your network | DICOM TLS, with optional PACS client certificates |
| Links between Guardian's own services on the Guardian host | Unencrypted, on the host only | — (see below) |

On Guardian Pro Cloud, DICOM and HL7 traffic runs inside the IPsec tunnel, which already encrypts it (see [Connecting your network](../cloud-onboarding.md#connecting-your-network)). If your policy also requires TLS inside the tunnel, ask Zauron.

Every option on request is off by default, so turning it on is a deliberate change that Zauron makes with you.

## DICOM over TLS

For a dedicated installation, Guardian can use DICOM TLS (DIMSE over TLS) in both directions.

- **TLS 1.2 or newer.** TLS 1.2 connections use ECDHE key exchange with AES-GCM or ChaCha20-Poly1305.
- **Guardian to your PACS**, set per PACS connection. Guardian always verifies the PACS certificate, against your CA certificate or the operating system's trust store; there is no "encrypt without verifying" mode. The PACS host name is verified too; when the PACS is addressed by IP, you tell Zauron the name its certificate carries. For mutual TLS, Guardian presents a client certificate you issue. Every association uses it: image queries and retrieves, the connection test in the Admin tab and the health check's echo. The DICOM TLS port is often `2762`; use the port your PACS listens on.
- **Your PACS to Guardian.** Guardian's DICOM receiver can accept TLS only on its port, or keep its plain port and add **one** extra fixed TLS port, so your PACS systems can move to TLS one at a time. It can require a client certificate from each PACS, issued by a CA you name.
- **Certificates** are PEM files on the Guardian host, given to Guardian read-only; they are never stored in the database. Renewed files are used by the next outbound association, and by the receiver after it restarts.
- **Fails closed.** If a TLS setting cannot be loaded, Guardian does not fall back to plain DICOM: the outbound association is not attempted, and the receiver does not start.

DataForge's own DICOM retrieve does not support DICOM TLS yet; it uses plain DICOM inside your network.

## HL7 MLLP over TLS

For a dedicated installation, Guardian's HL7 listener can accept MLLP over TLS 1.2 or newer, on every HL7 port or only the ports you choose.

- **Mutual TLS** (optional): Guardian requires a client certificate from your interface engine, issued by a CA you name, and can also check its subject name.
- The address check (only your interface engine's addresses may connect) still runs first.
- A plain connection to a TLS port is refused before any HL7 is read. If Guardian's certificate cannot be loaded, the listener refuses every connection, encrypted or not.
- Renewed certificate files are used by the next connection.

See the [HL7 Report Interface](../integrations/hl7.md) for the message profile.

## Links inside the Guardian host

Guardian's web entry point (the reverse proxy) is the only service that accepts connections from the network, and it accepts only HTTPS. Behind it, Guardian's own services talk to each other on the same host without TLS: the dashboard and its operator console, the viewer, DataForge, the report-search model and a short-lived cache. Each of these listens only on the host's loopback address or Docker's internal bridge network, so it cannot be reached from your network. The cache holds no data on disk and nothing survives a restart.

Why this is acceptable: only a process already running on the Guardian host could read this traffic, and such a process could equally read the data on disk, the configuration and process memory, so TLS between the services would not change the outcome. The database is different: it can be on another host, so that connection always uses TLS. If any of these services moves to another host, it will use TLS as well.

## Questions

Contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
