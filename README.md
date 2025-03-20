# Argo CD ApplicationSet and Helm Showcase

You can use it with the installed instance of Argo CD on your cluster.\
It creates namespaces based on the directory structure in this repo.\
Then it configure and install apps in these namespaces also based on config.

To initiate the process you must create the following `ApplicationSet` object in your Argo CD namespace.\
Because it uses apps-of-apps concept you must also override the Argo CD namespace if it differs from `argocd` using the `argoNamespace` parameter in the `spec.template.spec.sources[0].helm.parameters` section.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: global-config
spec:
  goTemplate: true
  generators:
    - git:
        repoURL: https://github.com/piomin/argocd-showcase.git
        revision: HEAD
        directories:
          - path: projects/*
  template:
    metadata:
      name: '{{.path.basename}}-config'
    spec:
      destination:
        namespace: '{{.path.basename}}'
        server: 'https://kubernetes.default.svc'
      project: default
      sources:
        - path: chart
          repoURL: 'https://github.com/piomin/argocd-showcase.git'
          targetRevision: HEAD
          helm:
            valueFiles:
              - $values/projects/{{.path.basename}}/values.yaml
            parameters:
              - name: appName
                value: '{{.path.basename}}'
        - repoURL: 'https://github.com/piomin/argocd-showcase.git'
          targetRevision: HEAD
          ref: values
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

## Articles

1. It shows how to use the **Kargo** tool to continuously promote application images between environments. Go to the `kargo` directory in the repo for the example. A detailed guide may be found in the following article: [Continuous Promotion on Kubernetes with GitOps](https://piotrminkowski.com/2025/01/14/continuous-promotion-on-kubernetes-with-gitops/)
2. It shows how to use Helm with Argo CD Application Set Git Files and Directory Generators to install many applications in the cluster in the standardized way. Go to the `appset-helm-demo` directory in the repo for the example. A detailed guide may be found in the following article: [The Art of Argo CD ApplicationSet Generators with Kubernetes](https://piotrminkowski.com/2025/03/20/the-art-of-argo-cd-applicationset-generators-with-kubernetes/)