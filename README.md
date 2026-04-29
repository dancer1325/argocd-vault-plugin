# argocd-vault-plugin
![Pipeline](https://github.com/argoproj-labs/argocd-vault-plugin/workflows/Pipeline/badge.svg)
![Code Scanning](https://github.com/argoproj-labs/argocd-vault-plugin/workflows/Code%20Scanning/badge.svg)
[![Go Report Card](https://goreportcard.com/badge/github.com/argoproj-labs/argocd-vault-plugin)](https://goreportcard.com/report/github.com/argoproj-labs/argocd-vault-plugin)
![Downloads](https://img.shields.io/github/downloads/IBM/argocd-vault-plugin/total?logo=github)
[![codecov](https://codecov.io/gh/argoproj-labs/argocd-vault-plugin/branch/main/graph/badge.svg?token=6Xr7V8AMTE)](https://codecov.io/gh/argoproj-labs/argocd-vault-plugin)

<img src="https://github.com/argoproj-labs/argocd-vault-plugin/raw/main/assets/argo_vault_logo.png" width="300">

- == Argo CD plugin
  - allows
    - retrieve secrets -- from -- >=1 Secret Management tools (HashiCorp Vault, IBM Cloud Secrets Manager, AWS Secrets Manager, etc.) a
    - inject the secrets | Kubernetes resources

- goal 
  - secret management | GitOps & Argo CD
    - WITHOUT relying | operator OR custom resource definition
    - ALTHOUGHT, it can be used | other Kubernetes resource
      - _Example:_ deployments, configMaps, ...

## Documentation
You can our full set of documentation at https://argocd-vault-plugin.readthedocs.io/


## Blogs
- [Solving ArgoCD Secret Management with the argocd-vault-plugin](https://itnext.io/argocd-secret-management-with-argocd-vault-plugin-539f104aff05)
- [Introducing argocd-vault-plugin v1.0!](https://itnext.io/introducing-argocd-vault-plugin-v1-0-708433294b2d)
- [How to Use HashiCorp Vault and Argo CD for GitOps on OpenShift](https://cloud.redhat.com/blog/how-to-use-hashicorp-vault-and-argo-cd-for-gitops-on-openshift)

## Presentations
- [Shh, It’s a Secret: Managing Your Secrets in a GitOps Way - Jake Wernette & Josh Kayani, IBM](https://youtu.be/7L6nSuKbC2c)
