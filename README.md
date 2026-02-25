# kubeit-flux-example

This repository uses [flux-bootstrap](https://github.com/dnvgl/service-mesh-chart/tree/master/charts/flux-bootstrap) to deploy a sample applications to a Kubernetes cluster.

`bootstrap/values.yaml` file contains environments for FluxCD to deploy.

On self-support tenant's repo add this repo to deploy examples:

```yaml
      repositories:
        repoURL: https://github.com/dnv-opensource/kubeit-flux-example.git
        targetRevision: main
        path: gitops/bootstrap
        autosync: true
```

## Deploy application using kind:helmrelease

Custom charts are defined at `charts` directory. Those helm charts usually use dependency helm chart to deploy applications.
Thanks to that, it is easy to control version of the dependency helm chart and update it when needed.

`dev/tenant2-flux` directory contains a sample application that is deployed using `kind: HelmRelease`.

## Test Flux sops

1. Create sops key in Azure Key Vault:

   ```bash
   az keyvault key create --name sops-key --vault-name kubeit-dev-kv-sh-we --protection software --ops encrypt decrypt
   ```

2. Grant access to the key for the managed identity used by the cluster (SOPS Managed Identity).
   Assign `get`, `encrypt`, and `decrypt` permissions for the key to SOPS security group: `Az_KubeIT_SOPS_Env_Dev`.

3. Retrieve the key URL for the created key in Azure Key Vault:

   ```bash
   az keyvault key show --name sops-key --vault-name kubeit-dev-kv-sh-we --query key.kid
   ```

4. Create file `.sops.yaml` to allow encrypting secrets using the created key in Azure Key Vault:

   ```yaml
   creation_rules:
     - path_regex: secret-decrypted.yaml
       key_groups:
         - azure_kv:
             # URL of the key in Azure Key Vault
             - "https://my-kv.vault.azure.net/keys/sops-key/1234567890abcdef"
   ```

5. Download sops binary from `https://github.com/getsops/sops/releases`.

6. Create a secret file `secret-decrypted.yaml` with the content you want to encrypt:

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: secret-basic-auth
     namespace: tenant2-sops-prod
   type: Opaque
   stringData:
     password: password-test
   ```

7. Encrypt the secret file using sops:

   ```bash
   sops encrypt --azure-kv https://my-kv.vault.azure.net/keys/sops-key/1234567890abcdef secret-decrypted.yaml > secret-encrypted.yaml
   ```

   and store encrypted file as `secret-encrypted.yaml` in the repository under `secrets` directory.

8. Deploy to cluster using FluxCD. `kind:Kustomization` should use **sops** provider.

9. Check if decrypted secret is present on cluster.
