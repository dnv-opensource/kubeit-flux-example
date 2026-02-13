# kubeit-flux-example

Testing SOPS with flux

SOPS (Secrets OPerationS) is a tool for managing and encrypting secrets directly within YAML, JSON, ENV, or INI files, using encryption providers like AWS KMS, GCP KMS, Azure Key Vault keys, etc.
This example demonstrates how to use Azure Key Vault keys with SOPS.

How to use it?
1. Before starting the development cluster, create a new branch in the kubeit-flux-example repository where you will place your changes.

2. In prod/repo.yaml and staging/repo.yaml change branch to your working branch.

3. In your dev cluster settings (voa-platform-infra repository), change the bootstrap_force_minimal_config flag to false. This ensures the cluster uses the appropriate voa-platform-apps/argocd/env-config/values-dev.yaml file instead of the minimal values file. Update the values-dev.yaml file to point to your new branch, and run the deploy-dev pipeline. This will trigger the deployment of Flux examples (a basic SOPS application should appear in ArgoCD, though it may initially show error events—ignore these until encryption and workload identity is configured).

4. Create User Assigned Managed Identity in the same subscription as cluster (name of managed identity must end with '-dev-tenant2' e.g. 'mi-dev-tenant2') and assign Role 'Managed Identity Contributor' to Az_KubeIT_AcrReader_Env_Dev security group.
   Get clientId of the newly created managed identity and update infra/base/service-account.yaml.

5. In the Azure Portal, create an example Key Vault to handle the key for encrypting and decrypting secrets in this example. Inside the Key Vault, generate a new key.

6. Grant the Key Vault Administrator role to your new managed identity for the newly created Key Vault (PIM may be required), and get/list permissions for secrets. This will allow SOPS to decrypt secrets using the key in the Key Vault.

7. Modify the .sops.yaml file to include the URL of the newly generated key in your Key Vault.

8. Install SOPS on your local machine by following the instructions at https://github.com/getsops/sops.

9. To encrypt secret.yaml files, use the following command (replace the placeholder values with the URL of your Key Vault key and the path to your secret.yaml file):
sops encrypt --azure-kv https://x.vault.azure.net/keys/sops-test-key/0000 prod/secret.yaml
This command will create an encrypted output that should replace the original secret.yaml file. It encrypts sensitive fields like stringData: password, making them inaccessible without the key.
Hint: You may need to run export AZURE_ADDITIONALLY_ALLOWED_TENANTS=* before encrypting.

Encrypted secret should look similar to this:
```yaml
apiVersion: v1
kind: Secret
metadata:
    name: secret-basic-auth
type: Opaque
stringData:
    password: ENC[AES256_GCM,data:G2UL3Yhh7o4Re6X2QC2Fqj7r,iv:eUkumfOtDJLCmarlNn0G56Ip/1lKKnU+Dzd3xFOeGEc=,tag:Es1JVQxPmi2mZ6Alo82Q1g==,type:str]
sops:
    kms: []
    gcp_kms: []
    azure_kv:
        - vault_url: https://sops-test-kv.vault.azure.net
          name: sops-test-key
          version: 60e20b99212641feac3f44005d06b182
          created_at: "2025-01-09T12:02:53Z"
          enc: ufltDLJNP11lHKUlcxwtYCQaXezmPHE4WkAYlqSKvAJRgOZf0LefmbO8GjC4OrSfvramCZSgPZ0c0DFDYM6rOnNKH9Y4owrgTQX4tGy8fuBl1XTjeSXTWWJuSN4uDeyIe2K5-C8ICsG3GnWY6ohgJsS3RLEMtTq0ohx1NjsCSNzEiG6Q26sFHyxgG2TFD3BrF-Hw4cBo64D3DrxnAIrVyh2_mwsjKrFaOdwmIeCIeVwskR3HQoom3v4va_yNijQDhr0UqDUa7GUsNirSm2dmMdknyD6pzoLSeLqDLoIaF3_OGdabCbVaX6wFhSCkCaVLogYdjyVgyY7Z_-2KGnmVzQ
    hc_vault: []
    age: []
    lastmodified: "2025-01-09T12:02:55Z"
    mac: ENC[AES256_GCM,data:J1/qburPlWFex2hpFh2Im09aXD8eQc6lkAuwickRdaC7CCRAeiNUHWhHNFzjs0KTtHLz9PE3Mz8rluZpHh40k4oSDn7QqJgRhhQnrLaU7frRFfT6SqLCwqR6n6y4t28bVoyC9CshwaKTxFpl+15SLDQ3HyoyfvS8xKc/m5d3F/w=,iv:dOfyQkToIluoCYPTdlUZF0GX/+1K+LaC/V6/qpaMIzw=,tag:Y+i7iGkbWYvfglhgKbAXeg==,type:str]
    pgp: []
    encrypted_regex: ^(stringData)$
    version: 3.9.2
```

10. After modifying the secret.yaml files, push your changes to your branch.
