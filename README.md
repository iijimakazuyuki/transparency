# transparency

Transparent Helm chart rendering kubernetes manifests from values.yaml.

This chart can be applicable only limited purpose.
Please read the usage below.

## Usage

Add Helm chart repository and install `transparency` chart.

```
helm repo add transparency https://iijimakazuyuki.github.io/transparency
helm install transparency transparency/transparency
```

Nothing is installed.
Create `values.yaml` as following:

```yaml
manifests:
- apiVersion: v1
  kind: Secret
  metadata:
    name: test
  stringData:
    test: test
```

Upgrade the installed Helm chart using the `values.yaml`.

```
helm upgrade transparency transparency/transparency -f values.yaml
```

You will see the `Secret` defined in `values.yaml`.

```
kubectl get secret test -o yaml
```

## Advanced Usage

You can use `transparency` chart via Argo CD without connecting my public Helm chart.

Argo CD can be installed by using Helm.

```
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd
```

Install `transparent-chart-hosted-repository` chart.

```
helm install transparent-chart-hosted-repository transparency/transparent-chart-hosted-repository
```

If you use your manifest repository for Argo CD, you can add manifests rendered by Helm as following:

```
helm template transparent-chart-hosted-repository transparency/transparent-chart-hosted-repository > /path/to/your/manifest/repository/transparent-chart-hosted-repository.yaml
```

Now `transparent` chart is hosted in kubernetes cluster.
Argo CD can use the Helm repository.
Create `application.yaml` as following:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: advanced-usage
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  destination:
    name: in-cluster
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  source:
    path: ''
    repoURL: http://transparent-chart-hosted-repository
    targetRevision: 1.0.0
    chart: transparency
    helm:
      values: |
        manifests:
        - apiVersion: v1
          kind: Secret
          metadata:
            name: advanced-test
          stringData:
            test: test
```

Apply `application.yaml`.

```
kubectl apply -f application.yaml
```

You will see the `Secret` defined in `application.yaml`.

```
kubectl get secret advanced-test -o yaml
```

## More Advanced Usage

You can use `transparency` chart to create resources in another kubernetes cluster via Argo CD without another manifest repository.

The following `Application` requires another manifest repository:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: requires-another-repository
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  destination:
    name: another-cluster
    namespace: default
  source:
    path: .
    repoURL: https://github.com/example/another-repository
    targetRevision: main
    directory:
      recurse: true
```

It may be common usage of Argo CD, but you can achieve something similar using `transparency` chart as following:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: more-advanced-usage
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  destination:
    name: another-cluster
    namespace: default
  source:
    path: ''
    repoURL: http://transparent-chart-hosted-repository # or https://iijimakazuyuki.github.io/transparency
    targetRevision: 1.0.0
    chart: transparency
    helm:
      values: |
        manifests:
        - apiVersion: v1
          kind: Secret
          metadata:
            name: more-advanced-test
          stringData:
            test: test
```

Note that it is prefer to install the application chart rather than this `transparent` chart.
When the application chart is not provided or only few manifests are required, this `transparent` chart can be applicable..

