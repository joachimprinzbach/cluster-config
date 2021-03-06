# cluster-config

[![App Status](https://argocd.baloise.dev/api/badge?name=apps)](https://argocd.baloise.dev/applications/apps)

This repository contains the configuration-as-code for [Argo CD](https://argoproj.github.io/argo-cd/) and all the apps managed by it.


## Applications managed by Argo CD
Every application has its own subfolder where the configuration is located. See the README.md files for a short explanation in every subfolder.

The following deployments are currently in the cluster, managed by ArgoCD:

| App Name          | Folder                         | Sync Status                         |
| ----------------- | ------------------------------ | ------------------------------------| 
| Argo CD           |[argocd](argocd)                |[![App Status](https://argocd.baloise.dev/api/badge?name=argocd)](https://argocd.baloise.dev/applications/argocd)                 |
| Rook Ceph         |[Rook Ceph](rook-ceph)          |[![App Status](https://argocd.baloise.dev/api/badge?name=rook-ceph)](https://argocd.baloise.dev/applications/rook-ceph)           |
| Sealed Secrets    |[Sealed Secrets](sealed-secrets)|[![App Status](https://argocd.baloise.dev/api/badge?name=sealed-secrets)](https://argocd.baloise.dev/applications/sealed-secrets) |
| Jenkins X         |[Jenkins X](jenkins-x)          |- |                                  
| Eclipse CHE       |[Eclipse CHE](eclipse-che)      |[![App Status](https://argocd.baloise.dev/api/badge?name=eclipse-che)](https://argocd.baloise.dev/applications/eclipse-che)|
| Strimzi Kafka Op  |[Strimzi](strimzi)              |[![App Status](https://argocd.baloise.dev/api/badge?name=strimzi)](https://argocd.baloise.dev/applications/strimzi)|
| Harbor            |[Harbor](harbor)                |[![App Status](https://argocd.baloise.dev/api/badge?name=harbor)](https://argocd.baloise.dev/applications/harbor)|
| Monocular         |[Monocular](monocular)          |[![App Status](https://argocd.baloise.dev/api/badge?name=monocular)](https://argocd.baloise.dev/applications/monocular)|
| Katafygio         |[Katafygio](katafygio)          |[![App Status](https://argocd.baloise.dev/api/badge?name=katafygio)](https://argocd.baloise.dev/applications/katafygio)|
| Guacamole         |[Guacamole](guacamole)          |[![App Status](https://argocd.baloise.dev/api/badge?name=guacamole)](https://argocd.baloise.dev/applications/guacamole)|
  
### Adding a new app
Every app needs to be referenced in the [values.yaml in the apps folder](apps/values.yaml).
See the [apps README.md](apps/README.md) for details.

### Secrets Management
Secrets like passwords or API tokens can be managed using the Kubernetes [secrets](https://kubernetes.io/docs/concepts/configuration/secret/) resource type. However, it is not a good idea to store those secrets in the repository since they are only base64 encoded. We are using [Bitnami Sealed Secrets](sealed-secrets) to encrypt those secrets.

## Argo CD General Setup
In order to start with the configuration and for this config repository to have any impact, we need to get Argo CD setup first.

Install into the Cluster by following the [Manual](https://argoproj.github.io/argo-cd/getting_started/#1-install-argo-cd)

You can also use the OC CLI and replace `kubectl` commands by `oc`.

For high availability use the following command instead of the normal install: 
`oc apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml`
For a documentation of the High Availability (HA) configuration see [this page](https://argoproj.github.io/argo-cd/operator-manual/high_availability/)

## Managing Argo CD with Argo CD
We are dogfooding - therefore managing Argo CD using... Argo CD! 
See the [docs](https://argoproj.github.io/argo-cd/operator-manual/declarative-setup/#manage-argo-cd-using-argo-cd) for an explanation of these git repositories content.
Especially the [argocd](argocd) subfolder contains the configuration for Argo CD itself.

Once the Argo CD app is running, we need to create a first app (Argo CD iteself) - so it knows where this repository is located.

- hit the "NEW APPLICATION" button in the Argo CD UI
- select "EDIT AS YAML"
- Insert the following yaml content and hit "CREATE"

```yaml
project: default
source:
  repoURL: 'https://github.com/baloise-incubator/cluster-config.git'
  path: apps
  targetRevision: HEAD
  helm:
    valueFiles:
      - "values.yaml"
destination:
  server: 'https://kubernetes.default.svc'
  namespace: argocd
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

## Faster sync with webhook

For faster synchronization you should add a webhook to the git repository. See the [docs](https://argoproj.github.io/argo-cd/operator-manual/webhook/) for an example.


