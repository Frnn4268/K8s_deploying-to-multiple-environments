# Kubernetes used for Deploying to Multiple Environments

## Overview

Deploying applications to different environments (staging, production, etc.) can be complex. This guide provides examples of how to manage this complexity using various tools such as Kustomize, Helm, and Kluctl, each with their own strengths and weaknesses. Additionally, we briefly mention other tools that might be useful depending on your specific needs.

## Project Structure

Here's an overview of the project's directory structure:

```plaintext
│   .gitignore
│   devbox.json
│   README.md
│   Taskfile.yaml
│
├───gcloud
│       flake.lock
│       flake.nix
│
├───helm
│   │   .gitignore
│   │   Taskfile.yaml
│   │
│   └───api-golang-helm-chart
│       │   .helmignore
│       │   Chart.yaml
│       │   values.production.yaml
│       │   values.staging.yaml
│       │   values.yaml
│       │
│       └───templates
│               Deployment.yaml
│               IngressRoute.yaml
│               Secret.yaml
│               Service.yaml
│               _helpers.tpl
│
├───kluctl
│   │   .kluctl.yaml
│   │   deployment.yaml
│   │   Taskfile.yaml
│   │
│   ├───.helm-charts
│   │   ├───https_cloudnative-pg.github.io
│   │   │   └───charts
│   │   │       └───cloudnative-pg
│   │   │           └───0.21.4
│   │   │               │   .helmignore
│   │   │               │   Chart.lock
│   │   │               │   Chart.yaml
│   │   │               │   LICENSE
│   │   │               │   README.md
│   │   │               │   values.schema.json
│   │   │               │   values.yaml
│   │   │               │
│   │   │               ├───charts
│   │   │               │   └───cluster
│   │   │               │       │   .helmignore
│   │   │               │       │   Chart.yaml
│   │   │               │       │   grafana-dashboard.json
│   │   │               │       │   README.md
│   │   │               │       │   README.md.gotmpl
│   │   │               │       │   values.schema.json
│   │   │               │       │   values.yaml
│   │   │               │       │
│   │   │               │       └───templates
│   │   │               │               NOTES.txt
│   │   │               │               sidecar-configmap.yaml
│   │   │               │
│   │   │               ├───monitoring
│   │   │               │       grafana-dashboard.json
│   │   │               │
│   │   │               └───templates
│   │   │                   │   config.yaml
│   │   │                   │   deployment.yaml
│   │   │                   │   monitoring-configmap.yaml
│   │   │                   │   mutatingwebhookconfiguration.yaml
│   │   │                   │   NOTES.txt
│   │   │                   │   podmonitor.yaml
│   │   │                   │   rbac.yaml
│   │   │                   │   service.yaml
│   │   │                   │   validatingwebhookconfiguration.yaml
│   │   │                   │   _helpers.tpl
│   │   │                   │
│   │   │                   └───crds
│   │   │                           crds.yaml
│   │   │
│   │   └───https_traefik.github.io
│   │       └───charts
│   │           └───traefik
│   │               └───20.8.0
│   │                   │   .helmignore
│   │                   │   Changelog.md
│   │                   │   Chart.yaml
│   │                   │   Guidelines.md
│   │                   │   LICENSE
│   │                   │   README.md
│   │                   │   values.yaml
│   │                   │
│   │                   ├───crds
│   │                   │       ingressroute.yaml
│   │                   │       ingressroutetcp.yaml
│   │                   │       ingressrouteudp.yaml
│   │                   │       middlewares.yaml
│   │                   │       middlewarestcp.yaml
│   │                   │       serverstransports.yaml
│   │                   │       tlsoptions.yaml
│   │                   │       tlsstores.yaml
│   │                   │       traefikservices.yaml
│   │                   │
│   │                   └───templates
│   │                       │   daemonset.yaml
│   │                       │   dashboard-hook-ingressroute.yaml
│   │                       │   deployment.yaml
│   │                       │   extra-objects.yaml
│   │                       │   gateway.yaml
│   │                       │   gatewayclass.yaml
│   │                       │   hpa.yaml
│   │                       │   ingressclass.yaml
│   │                       │   NOTES.txt
│   │                       │   poddisruptionbudget.yaml
│   │                       │   prometheusrules.yaml
│   │                       │   pvc.yaml
│   │                       │   service-hub.yaml
│   │                       │   service-internal.yaml
│   │                       │   service-metrics.yaml
│   │                       │   service.yaml
│   │                       │   servicemonitor.yaml
│   │                       │   tlsoption.yaml
│   │                       │   tlsstore.yaml
│   │                       │   _helpers.tpl
│   │                       │   _podtemplate.tpl
│   │                       │   _service-internal.tpl
│   │                       │   _service-metrics.tpl
│   │                       │   _service.tpl
│   │                       │
│   │                       └───rbac
│   │                               clusterrole.yaml
│   │                               clusterrolebinding.yaml
│   │                               podsecuritypolicy.yaml
│   │                               role.yaml
│   │                               rolebinding.yaml
│   │                               serviceaccount.yaml
│   │
│   ├───config
│   │       production.yaml
│   │       staging.yaml
│   │
│   ├───namespaces
│   │       Namespace.demo-app.yaml
│   │       Namespace.postgres.yaml
│   │
│   ├───services
│   │   │   deployment.yaml
│   │   │
│   │   ├───api-golang
│   │   │   │   deployment.yaml
│   │   │   │
│   │   │   ├───config
│   │   │   │       production.yaml
│   │   │   │       staging.yaml
│   │   │   │
│   │   │   └───manifests
│   │   │           Deployment.yaml
│   │   │           IngressRoute.yaml
│   │   │           Job.db-migrator.yaml
│   │   │           Middleware.yaml
│   │   │           Secret.db-migrator-password.yaml
│   │   │           Secret.yaml
│   │   │           Service.yaml
│   │   │
│   │   ├───api-node
│   │   │   │   deployment.yaml
│   │   │   │
│   │   │   ├───config
│   │   │   │       production.yaml
│   │   │   │       staging.yaml
│   │   │   │
│   │   │   └───manifests
│   │   │           Deployment.yaml
│   │   │           IngressRoute.yaml
│   │   │           Middleware.yaml
│   │   │           Secret.yaml
│   │   │           Service.yaml
│   │   │
│   │   ├───client-react
│   │   │   │   deployment.yaml
│   │   │   │
│   │   │   ├───config
│   │   │   │       production.yaml
│   │   │   │       staging.yaml
│   │   │   │
│   │   │   └───manifests
│   │   │           ConfigMap.yaml
│   │   │           Deployment.yaml
│   │   │           IngressRoute.yaml
│   │   │           Service.yaml
│   │   │
│   │   ├───load-generator-python
│   │   │   │   deployment.yaml
│   │   │   │
│   │   │   ├───config
│   │   │   │       production.yaml
│   │   │   │       staging.yaml
│   │   │   │
│   │   │   └───manifests
│   │   │           ConfigMap.yaml
│   │   │           Deployment.yaml
│   │   │
│   │   └───postgres
│   │       │   deployment.yaml
│   │       │
│   │       ├───config
│   │       │       production.yaml
│   │       │       staging.yaml
│   │       │
│   │       └───manifests
│   │               Cluster.yaml
│   │               Secret.yaml
│   │
│   └───third-party
│       │   deployment.yaml
│       │
│       ├───cloudnative-pg
│       │       helm-chart.yaml
│       │       helm-values.yaml
│       │       kustomization.yaml
│       │       Namespace.yaml
│       │
│       └───traefik
│               helm-chart.yaml
│               helm-values.yaml
│               kustomization.yaml
│               Namespace.yaml
│
├───kluctl-single-service
│   │   .kluctl.yaml
│   │   deployment.yaml
│   │   Taskfile.yaml
│   │
│   ├───config
│   │       production.yaml
│   │       staging.yaml
│   │
│   ├───namespaces
│   │       Namespace.demo-app.yaml
│   │
│   └───services
│       │   deployment.yaml
│       │
│       └───api-golang
│           │   deployment.yaml
│           │
│           ├───config
│           │       production.yaml
│           │       staging.yaml
│           │
│           └───manifests
│                   Deployment.yaml
│                   IngressRoute.yaml
│                   Job.db-migrator.yaml
│                   Middleware.yaml
│                   Secret.db-migrator-password.yaml
│                   Secret.yml
│                   Service.yaml
│
└───kustomize
    │   Taskfile.yaml
    │
    ├───base
    │   │   kustomization.yaml
    │   │
    │   ├───api-golang
    │   │       Deployment.yaml
    │   │       IngressRoute.yaml
    │   │       kustomization.yaml
    │   │       Secret.yaml
    │   │       Service.yaml
    │   │
    │   ├───api-node
    │   │       Deployment.yaml
    │   │       IngressRoute.yaml
    │   │       kustomization.yaml
    │   │       Secret.yaml
    │   │       Service.yaml
    │   │
    │   ├───client-react
    │   │       ConfigMap.yaml
    │   │       Deployment.yaml
    │   │       IngressRoute.yaml
    │   │       kustomization.yaml
    │   │       Service.yaml
    │   │
    │   ├───common
    │   │       kustomization.yaml
    │   │       Middleware.yaml
    │   │       Namespace.yaml
    │   │
    │   └───load-generator-python
    │           ConfigMap.yaml
    │           Deployment.yaml
    │           kustomization.yaml
    │
    ├───production
    │   │   kustomization.yaml
    │   │
    │   ├───api-golang
    │   │   │   kustomization.yaml
    │   │   │
    │   │   └───patches
    │   │           Deployment.yaml
    │   │           IngressRoute.replace-host.yaml
    │   │
    │   ├───api-node
    │   │   │   kustomization.yaml
    │   │   │
    │   │   └───patches
    │   │           Deployment.yaml
    │   │           IngressRoute.replace-host.yaml
    │   │
    │   ├───client-react
    │   │   │   kustomization.yaml
    │   │   │
    │   │   └───patches
    │   │           Deployment.yaml
    │   │           IngressRoute.replace-host.yaml
    │   │
    │   ├───common
    │   │       kustomization.yaml
    │   │
    │   └───load-generator-python
    │       │   kustomization.yaml
    │       │
    │       └───patches
    │               Deployment.yaml
    │
    └───staging
        │   kustomization.yaml
        │
        ├───api-golang
        │   │   kustomization.yaml
        │   │
        │   └───patches
        │           Deployment.yaml
        │           IngressRoute.replace-host.yaml
        │
        ├───api-node
        │   │   kustomization.yaml
        │   │
        │   └───patches
        │           Deployment.yaml
        │           IngressRoute.replace-host.yaml
        │
        ├───client-react
        │   │   kustomization.yaml
        │   │
        │   └───patches
        │           Deployment.yaml
        │           IngressRoute.replace-host.yaml
        │
        ├───common
        │       kustomization.yaml
        │
        └───load-generator-python
            │   kustomization.yaml
            │
            └───patches
                    Deployment.yaml
```

## Examples

### [Kustomize](https://github.com/kubernetes-sigs/kustomize)

Kustomize uses an overlay model to merge base YAML configurations with environment-specific patches, allowing for flexible configuration management.

- **Integration**: Built directly into `kubectl`.

- **Pros**: No extra tooling required.

- **Cons**: Can be cumbersome and verbose for specific types of patching.

Within the `kustomize` directory, you will find `base` configurations for each service along with `staging` and `production` overlays. These overlays patch the image tag and DNS environment postfix, making environment-specific adjustments straightforward.

### [Helm](https://helm.sh/)

Helm allows for defining "charts" that bundle Kubernetes resources and provide a customizable interface for deployment.

- **Pros**: Commonly used, provides powerful templating.

- **Cons**: Requires boilerplate code, which can feel heavy for some use cases.

In the `helm` directory, there is a Helm chart for the `api-golang` service. This chart includes a simple interface for setting the version tag and DNS environment postfix via a `values.yaml` file.

### [Kluctl](https://kluctl.io/)

Kluctl combines features from both Helm and Kustomize, offering templating, deployment diffs, and a GitOps controller.

- **Pros**: Combines the best features of Helm and Kustomize.

- **Cons**: Requires familiarity with both Helm and Kustomize concepts.

Within the `kluctl` directory, there is a full example (including third-party dependencies) of deploying the application to staging and production environments. This configuration is also used for a GitOps deployment.

## Additional Tools

### [Timoni](https://timoni.sh/)

Timoni uses CUE as its language, adding type safety and data validation. It enables more control over the application lifecycle and allows configuration to be shipped alongside the application container images as a single OCI artifact.

### [CDK8s](https://cdk8s.io/)

CDK8s enables defining Kubernetes resources using general-purpose programming languages like TypeScript, Python, Java, and Go. This opens up possibilities for dynamic resource generation and testing that are challenging with YAML.

### [Pulumi](https://www.pulumi.com/kubernetes/)

Pulumi also allows defining Kubernetes resources in general-purpose programming languages. It is useful if you are already using Pulumi for managing infrastructure as code, allowing you to manage Kubernetes deployments within the same framework.

## Conclusion

Each tool discussed here has its strengths and is suited to different use cases and levels of complexity. Choose the one that fits your project's needs and your team's expertise. Whether you prioritize integration, ease of use, or advanced features, there's a tool to help you manage your Kubernetes deployments effectively.
