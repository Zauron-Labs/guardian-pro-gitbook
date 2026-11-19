# Secrets in your cloud secret store

A dedicated installation in your own cloud account can keep its secrets in your own secret store, rotate them there, and give Zauron only the secret **names**. Guardian's configuration then holds a *reference* instead of the value. Guardian reads the value when each service starts, using the machine's own cloud identity (an EC2 instance profile, an ECS task role or an Azure managed identity). Zauron never needs read access to your store and never sees the values.

This is optional: a configuration without references works as before, with secrets in the installation's protected configuration file.

## Reference syntax

Any secret setting can hold one of these references:

| Store | Reference | Notes |
|---|---|---|
| AWS Secrets Manager | `aws-sm://<secret name or ARN>[#<json key>]` | Reads the current version. Region: the ARN's, else the configured AWS region |
| AWS Systems Manager Parameter Store | `aws-ssm://<parameter name or ARN>[#<json key>]` | Decrypts SecureString parameters with KMS. A name that starts with `/` gives three slashes: `aws-ssm:///guardian/prod/llm-api-key` |
| Azure Key Vault | `azure-kv://<vault>/<secret>[/<version>][#<json key>]` | `<vault>` is the vault name, or a full host name for sovereign clouds |

`#<json key>` reads one key of a JSON secret, so one secret can hold several values. For example, the database password can come straight from an RDS-managed secret: `aws-sm://rds!db-1234#password`. Each secret is fetched once per service start, however many keys are read from it.

Examples:

```
aws-sm://guardian/prod/db#password
aws-sm://guardian/prod/app#encryption_key
aws-ssm:///guardian/prod/llm-api-key
azure-kv://example-vault/db-password
```

## Which secrets

Zauron gives you the list of setting names for your installation. They cover:

- the database password;
- the encryption key that protects the integration secrets Guardian stores (reporting-system passwords, directory and single sign-on secrets);
- the salt used to hash patient and accession identifiers;
- the keys that sign sessions and viewer links, and the bootstrap administrator credential;
- the cache password, when one is set;
- the AI language model key, when the endpoint uses one;
- email credentials (your SMTP relay);
- the directory bind password and reporting-system credentials, when set in the configuration rather than in the Admin tab (those are stored encrypted);
- optional services: cloud credentials for AI model hosting, the DataForge GPU service key.

DICOM and HL7 use no passwords; their TLS keys are files on the host (see [Encryption in transit](encryption-in-transit.md)).

**The registry pull token stays Zauron's.** Zauron issues it for your installation and it defines which images your licence covers. You can keep it in your store, but Zauron creates and rotates it and gives you the new value.

## Permissions

**AWS.** Give the instance profile or task role read access to exactly those secrets. No static AWS keys are needed:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {"Effect": "Allow", "Action": "secretsmanager:GetSecretValue",
     "Resource": "arn:aws:secretsmanager:<region>:<account>:secret:guardian/prod/*"},
    {"Effect": "Allow", "Action": "ssm:GetParameter",
     "Resource": "arn:aws:ssm:<region>:<account>:parameter/guardian/prod/*"},
    {"Effect": "Allow", "Action": "kms:Decrypt", "Resource": "<your KMS key ARN>",
     "Condition": {"StringEquals": {"kms:ViaService": ["secretsmanager.<region>.amazonaws.com",
                                                       "ssm.<region>.amazonaws.com"]}}}
  ]
}
```

On EC2 with IMDSv2, set the instance metadata **hop limit to 2** so the role's credentials are reachable from inside containers:

```bash
aws ec2 modify-instance-metadata-options --instance-id <id> --http-put-response-hop-limit 2
```

**Azure.** Grant the VM's managed identity the **Key Vault Secrets User** role on the vault.

## Behaviour

- **Fails closed.** A reference that cannot be read stops that service at start, with a message that names the setting and the reason (for example `AccessDeniedException` or a missing JSON key), never the value. Values are never logged.
- **The viewer needs values.** The viewer component cannot read references yet. Guardian refuses to start it with a reference in place of a value, rather than use the reference text as a key. Its few secrets (database password, link and session signing keys, the encryption key and the cache address) are delivered as values: on Amazon ECS through the task definition's `secrets` (from Secrets Manager or Parameter Store), or with Docker Compose by your own tooling writing them on the host.
- **Check.** Zauron can run a check on the host that lists each secret setting as a working reference, a failing reference (with the reason) or a plain value.

## Rotation

1. Rotate the secret in your store (a new version under the same name).
2. Restart the services that read it. Each service reads the current version as it starts.
3. For the database password, use your store's alternating-users rotation, or restart within the rotation window, so connections opened with the old password are not cut.
4. The encryption key is a wrapping key, not a password: changing it makes every stored integration secret unreadable. Rotate it only with Zauron's re-encryption procedure.

## Questions

Contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
