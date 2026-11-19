# Container security

Guardian Pro runs as a set of Docker containers on the Guardian host. This page describes how they are locked down, and each place where Guardian needs more than that baseline, with the reason and what limits the risk.

## Baseline

Every long-running Guardian service:

- runs as a **non-root user**;
- has **all Linux capabilities dropped** and adds none back;
- runs with **no-new-privileges**, so no process inside can gain privileges;
- is **never privileged**.

Services that keep no state also run with a **read-only root filesystem**, writing only to small in-memory scratch areas.

AI model containers that Guardian starts get **no Linux capabilities**, no-new-privileges, are never privileged, never get access to Docker, publish their port on the host's loopback address only, and receive de-identified images. On AWS, the AI model tasks Zauron operates drop all capabilities as well.

Release images are [signed and pinned by digest](image-verification.md). Third-party images Guardian runs (the cache and the Docker API proxy) are pinned to exact versions.

## Documented exceptions

| Exception | Why it is needed | What limits the risk |
|---|---|---|
| **The main Guardian service uses the host's network** | Its DICOM receiver and HL7 listener must see each sending system's real address to check it against your allowed PACS and interface engines, and the ports they listen on can change without a redeploy. | Non-root, no capabilities, no-new-privileges. Internal service ports listen on loopback and Docker's internal network only, never on all interfaces. It listens only on ports 1024 and above, so it needs no special privilege. Inbound access is limited by your firewall or cloud security group. |
| **Starting AI model containers needs the Docker API** | Guardian pulls and starts licensed model containers on the host. | No application container gets the Docker socket. Access goes through a filtering proxy that refuses running commands in containers, image builds, volumes, secrets and system calls. The proxy listens on loopback or on an internal network that only the services that start models can join, and itself runs with a read-only root filesystem, no capabilities and no-new-privileges. Model containers get no capabilities and are never privileged. |
| **The host updater runs as root** | It recreates Guardian's own containers during an upgrade, so it runs on the host. | Upgrade jobs are data, not commands: only known Guardian services, only release version numbers, only images from your configured registry. Each upgrade is health-checked, rolled back on failure, and recorded in Zauron's platform audit log. Its configuration is root-owned and cannot be changed from Guardian. It can [verify image signatures](image-verification.md#verify-every-upgrade-on-your-guardian-host) before any change. |
| **Short start-up steps run as root** | Before the non-root services start, they hand the data directories and TLS certificate files on the host to the Guardian user, so existing installations upgrade in place. | No network, a fixed command, only the four file-ownership capabilities they need, no-new-privileges. They exit within seconds and are removed a day later. |
| **Some services have a writable root filesystem** | The main Guardian service, database backup, and DataForge write large temporary files (image conversion, uploads, backups). | Non-root, no capabilities, no-new-privileges. Image files are owned by root and are not writable by the service user outside its own working directories. A change could affect only that container, never the image or the host. |
| **AI model images run as root inside their container** | Many vendor model images are built to run as root and break otherwise. | No capabilities, no-new-privileges, never privileged, no Docker access, port on loopback only, de-identified input. |
| **The operator account on the Guardian host is in the `docker` group** | Zauron's operators run maintenance without `sudo`. | A system account for named Zauron operators only; access follows your host access policy (named SSH keys, VPN, multi-factor authentication). Containers never get this group. |

Zauron reviews this list whenever one of these components changes, and at least once a year.

## Questions

For the full container review, contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
