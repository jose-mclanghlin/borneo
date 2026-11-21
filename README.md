# Borneo

Borneo provides a reusable Helm chart for Kubernetes workloads and container images for Node.js and the PostgreSQL client.

## GitOps architecture

| Repository | Owns |
| --- | --- |
| `borneo` | Application charts, application values, shared Helm templates, and container build definitions |
| `gitops` | Argo CD Applications, destination clusters and namespaces, source revisions, sync policies, and infrastructure add-on configuration |

An Argo CD Application declares which Borneo revision, chart directory, and values file describe the desired state for its destination. Argo CD pulls that revision, renders the manifests, and compares them with the live resources. The Application's sync policy determines when changes are applied. [OpenGitOps principles](https://opengitops.dev/#gitops-principles) · [Argo CD best practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)

## Repository structure

| Path | Purpose |
| --- | --- |
| [helm/app](helm/app) | Shared chart, default values, and values schema |
| [helm/charts/caacupe](helm/charts/caacupe) | Caacupe chart with complete values for dev, QA, and production |
| [helm/charts/itaugua](helm/charts/itaugua) | Itaugua chart with its own `values.yaml` |
| [containers/node](containers/node/README.md) | Node.js image with OpenTelemetry instrumentation |
| [containers/psql](containers/psql/README.md) | PostgreSQL client image |
| [docs/NOTES.md](docs/NOTES.md) | Helm validation, release management, and rollback |
| [docs/aws](docs/aws/README.md) | External Secrets Operator, EKS, and IAM policy examples |

## Configure an application in Argo CD

Define the Application in `gitops/clusters/<env>/apps/<app>/app.yaml`. Keep the chart and its application values in Borneo. This example shows the `spec` fields for Caacupe QA. Replace `BORNEO_COMMIT_SHA` with the full commit SHA validated for that environment. Set the destination server for the intended cluster. The example uses the cluster where Argo CD runs.

```yaml
spec:
  source:
    repoURL: https://github.com/jose-mclanghlin/borneo.git
    targetRevision: BORNEO_COMMIT_SHA
    path: helm/charts/caacupe
    helm:
      valueFiles:
        - qa.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: caacupe
```

| Chart | Values selection |
| --- | --- |
| Caacupe | Set `helm.valueFiles` to one of `dev.yaml`, `qa.yaml`, or `prod.yaml`. Each file is a complete environment configuration. |
| Itaugua | Helm automatically loads the chart's `values.yaml`. |

All Caacupe files currently render resources into the `caacupe` namespace. If environments share a cluster, set a distinct `app.namespace.name` in each environment file and match it in the corresponding Application destination.

## Validate locally with Helm

Run these commands from the repository root. They render the chart locally for review before opening a pull request.

### Caacupe

Pick one of [dev.yaml](helm/charts/caacupe/dev.yaml), [qa.yaml](helm/charts/caacupe/qa.yaml), or [prod.yaml](helm/charts/caacupe/prod.yaml). Each file contains the full Caacupe configuration and sets its own `app.configMap.data.APP_ENV`. Pass the selected file explicitly with `-f`.

```sh
helm dependency build ./helm/charts/caacupe --skip-refresh
helm lint ./helm/charts/caacupe -f ./helm/charts/caacupe/dev.yaml
helm template caacupe ./helm/charts/caacupe -n caacupe -f ./helm/charts/caacupe/dev.yaml
```

### Itaugua

Helm automatically loads the chart's [values.yaml](helm/charts/itaugua/values.yaml).

```sh
helm dependency build ./helm/charts/itaugua --skip-refresh
helm lint ./helm/charts/itaugua
helm template itaugua ./helm/charts/itaugua -n itaugua
```

## Add an application chart

1. Create a directory under `helm/charts/` with a `Chart.yaml` that declares the shared chart as a dependency.

   ```yaml
   apiVersion: v2
   name: my-app
   version: 0.1.0
   appVersion: "1.0.0"
   dependencies:
     - name: app
       version: 0.0.1
       repository: "file://../../app"
   ```

2. Set the application's configuration under the top-level `app:` key. Use the [shared values](helm/app/values.yaml) and [schema](helm/app/values.schema.json) as a reference, and an existing application chart as a starting point. The image is set in `app.deployment.container.image`.
3. Build the dependency, lint the chart, and render it with each values file you plan to deploy.

## Chart versions

| Field in Chart.yaml | What it controls |
| --- | --- |
| `version` | The version of the Helm chart. Bump it when releasing changes to templates, values, or dependencies. |
| `appVersion` | The application version recorded as chart metadata. Set the container tag separately in `app.deployment.container.image`. |
| `dependencies[].version` | The version of the shared `app` chart. Update this field in each consuming chart when adopting a new shared chart version. |

Caacupe can use the same chart version across dev, QA, and production. Select the environment through the values file at deployment time.

## Promote and roll back changes

1. Open a pull request in Borneo for chart or values changes. Run `helm lint` and `helm template` for each affected environment and review the rendered changes.
2. Open a pull request in GitOps to point the dev Application at the reviewed Borneo commit. Verify its sync status and application health in Argo CD.
3. Promote the same tested chart revision through QA and production using each environment's values file. When promoting a container build, pin the same image digest in the environment files.
4. To roll back an Argo CD deployment, revert the GitOps change or restore the last known-good source revision in Git. Verify the resulting sync and health in Argo CD.

Keep credentials in a secret manager and reference them from the chart. Configure resource requests, limits, and health probes for workloads based on the application's needs. See the [External Secrets setup](docs/aws/README.md).

## Troubleshooting

```sh
kubectl get pods -n caacupe
kubectl logs -n caacupe deployment/caacupe-deployment
kubectl get events -n caacupe --sort-by=.lastTimestamp
```
