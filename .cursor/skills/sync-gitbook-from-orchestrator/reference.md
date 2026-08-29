# Orchestrator GT field map

Use this when diffing GitBook installation requirements against Terraform and runtime config. Paths are under `/home/kalclark/guardian_orchestrator`.

## Azure Terraform (`terraform/azure/`)

| Customer must provide | Variable / resource |
|----------------------|---------------------|
| Existing VNet | `existing_vnet_name`, `existing_vnet_resource_group` |
| Unused Guardian CIDR | `guardian_subnet_address_prefix` (default `10.0.100.0/24`) |
| PACS CIDRs | `pacs_cidr_blocks` |
| Reporting CIDRs | `report_system_cidr_blocks` |
| Admin SSH CIDRs | `admin_cidr_blocks` (public IP only if set) |
| VM size | `vm_size` default `Standard_D4s_v3` |
| SSH user | `vm_admin_username` — GitBook requires **`zauron`** (update Terraform if it still defaults to `guardian`) |
| OS disk | `os_disk_size_gb` default 64 |
| Data disk | `data_disk_size_gb` default 512, mount `/opt/guardian-data` |
| OS image | Ubuntu 22.04 LTS Jammy |
| DB | PostgreSQL Flexible Server 16, private DNS, `guardian_pro` |
| LLM | Default **Llama Scout** (`GUARDIAN_LLM_MODEL=llama-scout`) in the customer or Zauron tenant |
| ACR | `acr_login_server` default `zauron.azurecr.io` plus username/password |
| Inbound NSG | 22 (admin), 8090 (VNet), 4000 (PACS) |
| Outbound NSG | DICOM dest 11112 to PACS; report 443 and 1433; internet 80/443 |

## AWS Terraform (`terraform/aws/`)

| Customer must provide | Variable / resource |
|----------------------|---------------------|
| Existing VPC | `existing_vpc_id` |
| VM subnet | `existing_subnet_id` |
| RDS subnets | `existing_db_subnet_ids` (2+ AZs) |
| Instance type | default `m5.xlarge` (4 vCPU / 16 GB) |
| LLM | `llm_endpoint` / `llm_api_key` placeholders; Bedrock not provisioned |
| Extra PACS egress | 11112 and 104 |

## Runtime

| Item | Source |
|------|--------|
| Env keys | `app/docker/.env.customer.template` |
| Full stack | orchestrator + Traefik + viewer compose |
| SCP/SCU | `GUARDIAN_LOCAL_SCP_AE_TITLE=GUARDIAN_SCP`, port 4000; `GUARDIAN_SCU`; SCU ports 11200–12000 |
| Reports | `report_client_powerscribe_api`, `report_client_db`, `report_client_fhir`, `report_client_dicom_sr` |
| LLM | Default **Llama Scout** (`llama-scout`); `GUARDIAN_LLM_PROVIDER=AZURE\|AWS` in code today |

## Do not copy into GitBook

Passwords, ACR tokens, OpenAI keys, `terraform.tfvars`, hash salts, JWT secrets.
