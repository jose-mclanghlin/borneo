# Helm operations

Run these commands from the repository root and select the Kubernetes context for the intended environment.

## Prepare and validate Caacupe

```sh
helm dependency build ./helm/charts/caacupe --skip-refresh
helm show values ./helm/app
helm lint ./helm/charts/caacupe -f ./helm/charts/caacupe/qa.yaml
helm template caacupe ./helm/charts/caacupe -n caacupe -f ./helm/charts/caacupe/qa.yaml
```

Use `dev.yaml`, `qa.yaml`, or `prod.yaml` to select the full configuration for an environment. For Itaugua, use `./helm/charts/itaugua` and omit `-f` to load its `values.yaml`. Rendered manifests can contain Secrets, so keep the output local.

## Inspect a release

```sh
helm list -n caacupe
helm status caacupe -n caacupe
helm history caacupe -n caacupe
kubectl get deployments,pods,services -n caacupe
kubectl describe deployment caacupe-deployment -n caacupe
kubectl logs deployment/caacupe-deployment -n caacupe
kubectl get events -n caacupe --sort-by=.lastTimestamp
```

For a release managed directly by Helm, choose a revision from `helm history` and run `helm rollback caacupe REVISION -n caacupe`. To remove it, run `helm uninstall caacupe -n caacupe`.

## Inspect an Argo CD Application

```sh
kubectl get applications.argoproj.io -n argocd
kubectl describe application caacupe -n argocd
```

Check the desired source revision, last synced revision, health, and sync errors. Adjust the Application name and namespace to match its definition in GitOps. Roll back by restoring the last known-good revision in GitOps, then check sync and health again.

See the [Argo CD configuration example](../README.md#configure-an-application-in-argo-cd) and the [AWS Secrets Manager setup notes](aws/README.md).
