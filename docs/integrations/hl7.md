# HL7 Report Interface

For the customer's **interface team** (interface engine / RIS / reporting-system analysts). This document is the interface contract for sending finalized radiology reports to Guardian over HL7 v2. It covers transport, acknowledgements, message content, report status handling, time zones, testing and go-live. It does not cover what Guardian does with a report after it is stored.

---

## 1. Summary

| Item | Value |
|------|-------|
| Direction | Your interface engine **sends** to Guardian (Guardian listens; you open the connection) |
| Framing | MLLP (start block `0x0B`, end block `0x1C 0x0D`) |
| Message | `ORU^R01`, HL7 v2.3, 2.3.1, 2.4, 2.5 or 2.5.1 |
| Acknowledgement | Original-mode ACK: `MSA-1` = `AA` / `AE` / `AR`, `MSA-2` = your `MSH-10` |
| Delivery guarantee | `AA` is sent only after the message is committed to durable storage |
| Connections | Persistent connections are fine; one message in flight at a time (send, wait for the ACK, send the next) |
| Network | Site-to-site VPN (standard), or Direct over the internet with mutual TLS and a source-IP allow-list |
| Endpoint | Host / IP and TCP port assigned by Zauron at onboarding (one port per customer) |

---

## 2. Transport

### 2.1 VPN (standard)

- Your network connects to Zauron's VPN gateway with an IPsec site-to-site tunnel (see [Connecting your network](../saas-onboarding.md#connecting-your-network)). On a [dedicated installation](../installation.md), Guardian runs inside your own network and no VPN is needed.
- Your interface engine connects to the Guardian HL7 address and port Zauron gives you, through the tunnel.
- Plain MLLP inside the tunnel (the tunnel already encrypts the traffic). No TLS is needed.
- Only your agreed address range may connect. A connection from any other address is closed without an ACK.

### 2.2 Direct (no VPN): mutual TLS + source-IP allow-list

Use Direct only when a VPN is not possible. The HL7 port then **never accepts plaintext**.

- Your engine connects over the internet to `<your Guardian hostname>:<port>` (Zauron provides both).
- TLS 1.2 or newer. Guardian presents a publicly trusted server certificate for your Guardian hostname; verify it (hostname check on).
- Your engine **must present a client certificate**. Guardian accepts it only if it is signed by a CA you gave us and, when you ask for it, only if its subject CN / SAN matches the name(s) you gave us.
- The connection must come from one of your registered public IP addresses. Anything else is closed before the TLS handshake.
- A missing, expired or unrecognized client certificate, plaintext, or an unlisted source address all close the connection without an ACK.

**You provide for Direct mode:**

| What | Notes |
|------|-------|
| CA certificate(s) (PEM) that sign your engine's client certificate | The CA only: never the client certificate's private key. Intermediate + root is fine. |
| Optional subject pin(s) | Client certificate CN or SAN value(s) to accept (e.g. `hl7-engine.hospital.org`) |
| Public source IP addresses / ranges | The addresses your engine's traffic leaves your network from (after NAT). Use the narrowest ranges possible. |
| Certificate renewal contact | Send the new CA before it changes; client certificates signed by the same CA need no change on our side. |

### 2.3 Connection behaviour

- Keep one persistent connection open, or connect per message: both work.
- An idle connection is closed after **10 minutes** without traffic (agreed value; can be changed). Your engine should reconnect automatically when it has a message to send.
- Once a message starts, it must arrive completely within **60 seconds**.
- Maximum message size: **16 MB**.
- Send **one message at a time** and wait for its ACK before sending the next. This keeps preliminary → final → corrected versions of a report in order.

---

## 3. Acknowledgements and retries

Guardian answers every complete message with an original-mode `ACK` (`MSH-9` = `ACK^R01`), `MSA-2` echoing your `MSH-10`.

| `MSA-1` | Meaning | What your engine should do |
|---------|---------|-----------------------------|
| `AA` | Accepted. The message is committed to durable storage **before** this ACK is sent. | Send the next message. |
| `AE` | Rejected: the message cannot be used as sent (e.g. no accession number, not an ORU, unreadable). `MSA-3` says why. | Do **not** resend the same message unchanged. Queue it for your analyst, fix the content, resend with a new `MSH-10`. |
| `AR` | Temporary failure: the message could not be stored. | Resend the **same** message (same `MSH-10`) after a short back-off (e.g. 30 s, then 1, 2, 5 minutes). |
| no ACK | Connection dropped or timed out before the ACK. | Reconnect and resend the same message (same `MSH-10`). |

**Resends are safe.** A message whose `MSH-10` Guardian has already stored is acknowledged `AA` again and not stored twice. For the same reason, every *new* message (including each preliminary, final and corrected version of a report) needs its own unique `MSH-10`.

---

## 4. Message structure

```
MSH
[PID]
{ OBR            one OBR per accession number
  [{NTE}]
  { OBX          report text
    [{NTE}] }
}
```

Other segments (`PV1`, `ORC`, `SPM`, `ZXX` …) may be present and are ignored.

Standard encoding characters `|^~\&` are expected. Character set: `MSH-18` is honoured (`UNICODE UTF-8`, `8859/1`, `8859/15`, `ASCII`, …); when `MSH-18` is empty the message is read as UTF-8.

### 4.1 Field requirements

R = required (message is rejected `AE` without it or unusable), RE = required if known (send it whenever your system has it), O = optional, — = not used.

| Field | Name | Req | Content and notes |
|-------|------|-----|-------------------|
| MSH-3 | Sending application | RE | Your system's name, agreed at onboarding (e.g. `RIS`, `PS360`). |
| MSH-4 | Sending facility | RE | Your facility code. |
| MSH-5 | Receiving application | O | `GUARDIAN` unless another value is agreed. |
| MSH-6 | Receiving facility | O | Agreed value, or empty. |
| MSH-7 | Date/time of message | R | `YYYYMMDDHHMMSS[±ZZZZ]`. See §5 for time zones. |
| MSH-9 | Message type | R | `ORU^R01` (`ORU^R01^ORU_R01` is fine). `ACK` and other types are rejected. |
| MSH-10 | Message control ID | R | Unique per message, including each P / F / C version of a report. Used for duplicate detection. |
| MSH-11 | Processing ID | O | `P` in production. |
| MSH-12 | Version ID | R | `2.3`, `2.3.1`, `2.4`, `2.5` or `2.5.1`. |
| MSH-18 | Character set | O | See above; default UTF-8. |
| PID-3 | Patient identifier list | R | The medical record number. With several repetitions, the one whose identifier type (CX-5) is `MR` is used, else the first. |
| PID-5 | Patient name | — | Not used. You may omit it or send it as your engine requires. |
| PID-7 | Date of birth | RE | `YYYYMMDD` (year or year+month alone are accepted). Used to derive the patient's age at the exam date; no age is recorded without it. |
| OBR-2 | Placer order number | O | Used as the accession only when OBR-3 is empty. |
| OBR-3 | Filler order number | R | **The accession number.** One OBR per accession. |
| OBR-4 | Universal service ID | R | `code^description[^coding system]`, e.g. `74177^CT ABDOMEN PELVIS W CONTRAST^CPT`. The procedure description should match what the reporting system shows. |
| OBR-7 | Observation date/time | R | Exam date/time. Required for age at exam and report timing. |
| OBR-25 | Result status | R | `F` final, `C` corrected (`A` amended is treated as a correction), `P` preliminary, `X` cancelled. See §6. Blank is treated as final. |
| OBR-32 | Principal result interpreter | RE | **The signing radiologist**: `ID&Last&First` (v2.3+ NDL form) or `ID^Last^First`. The ID must be the radiologist identifier your reporting system uses, so reports are attributed to the right person. If OBR-32 is empty, OBR-16 is used. Reports with neither are **accepted** but attributed to an *unknown radiologist* placeholder until corrected. |
| OBX-2 | Value type | R | `TX` or `FT` (`ST` accepted). OBX segments with other value types (`ED`, `RP`, `NM`, …) are ignored. |
| OBX-3 | Observation identifier | O | Any (e.g. `&GDT^REPORT`, `IMP^IMPRESSION`). |
| OBX-5 | Observation value | R | The report text. Either one OBX per line, or one `FT` value with `\.br\` line breaks; repetitions (`~`) are separate lines. Standard escapes (`\F\ \S\ \T\ \R\ \E\ \Xhh\ \.br\ \.sp\`) are decoded. Send the **complete** report text (findings, impression, addenda). |
| OBX-11 | Observation result status | O | Not used; OBR-25 is authoritative. |
| NTE-3 | Comment | O | Appended after the report text. |

### 4.2 Reports covering several accessions

When one dictated report covers several exams (e.g. CT chest and CT abdomen/pelvis read together), send **one** ORU with **one OBR per accession** and the shared report text once (the OBX segments may follow the first OBR). Do not send separate messages per accession with the same text: the exams would not be linked.

---

### 4.3 Example (final report, two accessions)

```
MSH|^~\&|RIS|MAINHOSP|GUARDIAN||20260915143210-0500||ORU^R01|RIS000123457|P|2.5.1||||||UNICODE UTF-8
PID|1||00123456^^^MAINHOSP^MR||TEST^PATIENT||19650412|F
OBR|1|ORD5551|ACC20260915001|71260^CT CHEST W CONTRAST^CPT|||20260915131500-0500||||||||||||||||||F|||||||1234567&Radiologist&Pat
OBR|2|ORD5552|ACC20260915002|74177^CT ABDOMEN PELVIS W CONTRAST^CPT|||20260915131500-0500||||||||||||||||||F|||||||1234567&Radiologist&Pat
OBX|1|FT|&GDT^REPORT||EXAM: CT CHEST, ABDOMEN AND PELVIS WITH CONTRAST\.br\\.br\FINDINGS: ...\.br\\.br\IMPRESSION: ...||||||F
```

(Synthetic values. Use test patients during testing, see §7.)

---

## 5. Time zones

- Timestamps **with an offset** (`YYYYMMDDHHMMSS-0500`) are preferred and always read exactly.
- Timestamps **without an offset** are read as local time in your reporting system's time zone, which you give us at onboarding (it can differ per report system). Tell us if your system sends UTC without an offset.
- Daylight-saving transitions are handled from the IANA zone (e.g. `America/Chicago`), so give us the zone name, not a fixed offset.

---

## 6. Report status handling

| OBR-25 | Handling |
|--------|----------|
| `P` preliminary | Stored and **held**. It is not used until the final version of the same accession arrives. |
| `F` final | Used. This is the version of record. |
| `C` corrected / `A` amended | **Replaces** the previous final for the same accession(s). Send the **complete** corrected report text (not only the change), with a new `MSH-10` and the same accession number(s). Addenda are sent this way too. |
| `X` cancelled | That OBR's accession is not attached to the report. A message whose OBRs are all cancelled is stored for audit only. |
| `I`, `R`, `S`, `O` (other) | Stored for audit, not used. |
| blank | Treated as `F`. |

Send versions in the order they occur (P, then F, then any C) and wait for each ACK (§2.3). If your system can only send finals, send only `F` and `C`: that is fully supported.

---

## 7. Test-message checklist

Run these in the test phase, using **test patients** where your system allows it. Zauron confirms each result on its side.

| # | Test | Expected |
|---|------|----------|
| T1 | Connect from each interface engine address (VPN: through the tunnel; Direct: TLS with your client certificate) | Connection accepted |
| T2 | Direct only: connect from an unlisted address, without a client certificate, and in plaintext | All three refused, no ACK |
| T3 | One final report, one accession | `AA`; Zauron confirms accession, radiologist, exam date and text |
| T4 | Resend T3 unchanged (same `MSH-10`) | `AA`; no duplicate on Guardian |
| T5 | Preliminary, then final, same accession (different `MSH-10`s) | `AA`, `AA`; only the final is used |
| T6 | Final, then corrected (full text) | `AA`, `AA`; the corrected text replaces the final |
| T7 | One report covering two accessions (two OBRs) | `AA`; both exams linked to the one report |
| T8 | Final with OBR-32 (and OBR-16) empty | `AA`; attributed to the unknown-radiologist placeholder |
| T9 | Message with no accession (OBR-2 and OBR-3 empty) | `AE` with a reason in `MSA-3` |
| T10 | Timestamps with and without an offset | Exam and report times match your system in your local time |
| T11 | Special characters (`& ^ ~ \ |`), non-ASCII names in the text, a long multi-page report | Text round-trips exactly |
| T12 | Leave the connection idle past the idle timeout, then send | Engine reconnects and the message gets `AA` |
| T13 | Radiologist ID check: one report from each reading radiologist (or a staff list with their IDs) | Every ID maps to a known radiologist |

---

## 8. Go-live criteria

Guardian switches your HL7 feed to production when all of these hold:

1. **All test cases T1–T13 pass** (T2 applies to Direct mode only).
2. **Parallel run**: the live feed runs for at least **5 business days** alongside the current report source (Guardian receives and stores, but does not use, the HL7 reports), and Zauron's comparison shows:
   - every final report in the period was received over HL7 (no missing accessions);
   - radiologist, patient age and exam date match the current source for every report (differences explained and fixed);
   - preliminary / final / corrected versions and multi-accession reports behave as in §4.2 and §6;
   - no unexplained `AE`, and every `AR` recovered by resend.
3. **Radiologist IDs**: every active reading radiologist's OBR-32 ID is known (no reports attributed to the unknown placeholder).
4. **Time zone** confirmed (T10).
5. **Monitoring agreed**: Zauron is alerted when your feed is silent for longer than an agreed period (default 4 hours) or when rejected / failed messages spike. Give us a 24×7 contact for the interface engine and your planned-downtime notice process.
6. **Cut-over plan**: the date the previous report source stops being used, and who confirms the first production day.

---

## 9. Contacts and changes

- Tell Zauron **before** changing: engine IP addresses, client CA (Direct), sending application / facility values, HL7 version, character set, or the time zone your system sends.
- Planned engine downtime: notify Zauron in advance so the silence alert is expected. Queued messages sent after the downtime are accepted in order.
- Questions about this profile: your Zauron onboarding contact.
