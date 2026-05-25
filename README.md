# crossplane-package-storage-bucket

Crossplane Configuration package multi-cloud para provisioning de buckets
object storage con credenciales: AWS S3 + OCI Object Storage S3-compat.

Split en 3 sub-paquetes para optimizar instalación por cluster:

| Sub-paquete | Contenido | Instalar en |
|---|---|---|
| `base/` | XRD `XStorageBucket` + deps comunes (function-kcl, function-env-configs, provider-kubernetes) | (todos via dependsOn) |
| `aws/` | Composition `storage-bucket-aws` + deps AWS providers (family-aws, aws-iam, aws-s3 v1.21.1) | clusters cloud=aws |
| `oci/` | Composition `storage-bucket-oci` + deps OCI provider (oci-objectstorage v1.1.0) | clusters cloud=oci |

Cada cluster instala UN solo paquete cloud-specific. Las deps se resuelven
automáticamente vía dependsOn del paquete (incluido el `-base` con el XRD).

## Versionado

Los 3 sub-paquetes se publican con la misma versión semver. Pipeline en
`workflows/release.yaml` (Argo Workflow submitible vía CLI):

```bash
argo --kube-context control-ireland -n argo-workflows submit \
    packages/crossplane-package-storage-bucket/workflows/release.yaml \
    -p branch=main
```

Tags `vX.Y.Z` (main → "stable" channel) o `vX.Y.Z-int.N` (prerelease para
canary in non-live envs).

## Uso (XR claim)

```yaml
apiVersion: storage.docline.io/v1alpha1
kind: XStorageBucket
metadata:
  name: docline-storage
  namespace: infra-int   # env derivado del namespace prefix "infra-"
spec:
  bucketName: x-docline-storage   # bucket cloud final = "x-docline-storage-int"
  publishToInfisical: true        # push AWS_BUCKET/ENDPOINT/REGION/KEY/SECRET → Infisical
  infisicalProject: core-secrets
  infisicalPath: /storage
```

La Composition deriva `cloud` + `region` del `EnvironmentConfig cluster-platform`
del cluster destino y emite los Managed Resources correspondientes.

## Branches (Kargo)

- `main`: stable channel
- `int`, `qa`, `staging`, `prod-morocco`: cada branch = un env. Kargo
  promueve commits main → int → qa → staging → prod-morocco según Stage
  definitions en core-repo Flow XR.
