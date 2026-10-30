# `blackbox-operator`

A [Helm Operator](https://github.com/operator-framework/helm-operator) that installs and manages
[Prometheus Blackbox Exporter](https://github.com/prometheus/blackbox_exporter) Helm charts inside a
Kubernetes cluster.

The operator watches the custom resources defined in `watches.yaml` and reconciles them into running
Helm releases, so Blackbox Exporter and its probe configuration can be managed declaratively rather
than by running `helm install` by hand.

## Contents

| Path | Description |
| ---- | ----------- |
| `watches.yaml` | Operator watch definitions mapping CRDs to Helm charts. |
| `build/Dockerfile` | Builds the operator image from `quay.io/operator-framework/helm-operator:v0.13.0`. |
| `helm-charts/blackbox/` | Helm chart for the Blackbox Exporter (`prometheus-blackbox-exporter`). |
| `helm-charts/blackbox-check/` | Helm chart for Blackbox `BlackboxCheck` / probe resources. |
| `deploy/` | Raw manifests: CRDs, the operator Deployment, RBAC and ServiceAccount. |
| `helm-charts/blackbox/README.md` | Chart-level documentation for the Blackbox Exporter chart. |

## Custom resources

The operator serves the `blackbox.monitoring.com` API group:

| Kind | Group / version | Chart |
| ---- | -------------- | ----- |
| `Blackbox` | `blackbox.monitoring.com` / `v1alpha1` | `helm-charts/blackbox` |
| `BlackboxCheck` | `blackbox.monitoring.com` / `v1alpha1` | `helm-charts/blackbox-check` |

CRD definitions and example custom resources are in `deploy/crds/`:

- `blackbox.monitoring.com_blackboxes_crd.yaml`
- `blackbox.monitoring.com_blackboxchecks_crd.yaml`
- `blackbox.monitoring.com_v1alpha1_blackbox_cr.yaml`
- `blackbox.monitoring.com_v1alpha1_blackboxcheck_cr.yaml`

## Building the operator image

```bash
docker build -f build/Dockerfile -t blackbox-operator:latest .
```

The image layers the operator binary with `watches.yaml` and the `helm-charts/` directory, which is
where the Helm Operator expects to find its watch configuration and chart sources.

## Deploying

Apply the raw manifests directly:

```bash
kubectl apply -f deploy/crds/
kubectl apply -f deploy/service_account.yaml
kubectl apply -f deploy/role.yaml
kubectl apply -f deploy/role_binding.yaml
kubectl apply -f deploy/operator.yaml
```

Or install the charts with Helm:

```bash
helm install blackbox helm-charts/blackbox
helm install blackbox-check helm-charts/blackbox-check
```

Once the operator is running, create a `Blackbox` resource and it will be reconciled into a Helm
release automatically.

## Configuration

The Blackbox Exporter chart is configured through its `values.yaml`, and probe definitions are
supplied through `BlackboxCheck` resources. Refer to
[`helm-charts/blackbox/README.md`](helm-charts/blackbox/README.md) for the full list of available
values.

## License

MIT — see [`LICENSE`](LICENSE).
