---
name: sync-gitbook-from-orchestrator
description: >-
  Aligns Guardian Pro GitBook with guardian_orchestrator ground truth, including
  Terraform modules, Docker Compose, .env.customer.template, and the live/dev
  deployment shape. Use when updating GitBook, reviewing installation
  requirements, comparing docs to Terraform/NSG/ports/LLM/PACS/reporting, or
  when the user mentions guardian_orchestrator, GitBook drift, or live dev env.
---

# Sync GitBook from Orchestrator GT

Guardian Pro customer docs live in this repo. **Ground truth is** `/home/kalclark/guardian_orchestrator`, not GitBook and not remembered chat.

## When to run

- User asks to update GitBook, installation requirements, network diagrams, or APIs
- User asks whether docs match Terraform, Docker, or the live/dev environment
- User mentions guardian_orchestrator as the source of truth

## Ground-truth sources (read in this order)

1. **Terraform (what cloud customers must provide)**
   - `/home/kalclark/guardian_orchestrator/terraform/azure/`
   - `/home/kalclark/guardian_orchestrator/terraform/aws/`
   - Start with `variables.tf`, `network.tf`, `compute.tf`, `database.tf`, `openai.tf` (Azure), `terraform.tfvars.example`
2. **Runtime config**
   - `/home/kalclark/guardian_orchestrator/app/docker/.env.customer.template`
   - `/home/kalclark/guardian_orchestrator/app/docker/DEPLOYMENT_GUIDE.md`
   - Compose: `docker-compose.orchestrator.yml`, `docker-compose.traefik.yml`, `docker-compose.viewer.yml`, `docker-compose.guardian_full_stack.yml`
3. **Product behavior**
   - PACS: `app/src/pacs/PACS_README.md`
   - Reports: `app/src/report_ingestion/REPORT_INGESTION_README.md`
   - LLM: `app/src/ai_processing_llm/AI_LLM_README.md`
   - CV/models: `app/src/ai_processing_cv/AI_CV_README.md`
   - Dashboard: `app/src/informatics/INFORMATICS_README.md`
4. **Live/dev check (only if reachable; never print secrets)**
   - Compose health, published ports, AE titles, LLM provider, DB version
   - Compare **key names and topology**, not credential values
   - If the VM is not reachable, Terraform + `.env.customer.template` are sufficient GT

Do not treat `app/docker/DOCKER_README.md` "example Terraform outputs" as GT when they disagree with the actual `terraform/` modules.

## GitBook targets

Edit `/home/kalclark/guardian-pro/docs/` (especially `installation.md`, `installation-checklist.csv`, `api_reference.md`, `models.md`, `options.md`). Keep `SUMMARY.md` in sync if pages are added.

## Workflow

```
Task progress:
- [ ] Read Terraform variables/network/compute/database/LLM
- [ ] Read .env.customer.template + compose ports/services
- [ ] Diff against docs/installation.md requirements and checklist
- [ ] Diff APIs/models only if those pages are in scope
- [ ] Patch GitBook; do not invent installer URLs or retired APIs
- [ ] Keep customer-facing tone (no internal compose filenames unless needed)
```

1. Extract current GT: VM size, disks, OS, SSH user, VNet/VPC inputs, NSG/SG ports, DB version/access, LLM provider/tenant, ACR, PACS AE titles/ports, report clients.
2. Compare to GitBook requirements. Flag mismatches as **missing**, **stale**, or **over-claimed** (docs say it; code/Terraform do not).
3. Update GitBook to match GT. If product intent (user) conflicts with code, record the conflict in the reply and prefer explicit user instruction for customer-facing claims.
4. Do not commit unless asked. Never copy passwords, API keys, or `.tfvars` values into GitBook.

## Over-claim guardrails

| Topic | GT today |
|-------|----------|
| Terraform clouds | Azure and AWS modules; Google and Self-hosted are documented as customer-provisioned |
| LLM | Default **Llama Scout** (`llama-scout`); customer tenant or Zauron tenant |
| Default Azure VM | `Standard_D4s_v3` (4 vCPU / 16 GB); data disk 512 GB |
| SSH user | **`zauron`** (GitBook GT). Terraform may still default to `guardian` until that repo is updated |
| Network | Deploy **into existing** VNet/VPC; collect PACS, report, and admin CIDRs |
| PostgreSQL | Terraform provisions 16, private, SSL, `uuid-ossp` + `pgcrypto` |
| Report clients | PowerScribe API, SQL (`1433`), FHIR, DICOM SR; HL7 v2 if product says so |
| PACS AE titles | `GUARDIAN_SCP` / `GUARDIAN_SCU`; SCP port `4000` |
| Registry | `zauron.azurecr.io` |
| Retired | `/api/v1/pacs/events/*` (do not document as live) |
| Traefik | Product needs inbound `80`/`443` even if a given NSG snapshot omits them |

## Reply format

Lead with what changed in GitBook, then remaining GT conflicts (code vs docs vs Terraform) in a short table.

## Additional resources

- Field map: [reference.md](reference.md)
