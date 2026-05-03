# argocd-vault-plugin

- goal
  - secret management | GitOps & Argo CD
    - WITHOUT relying on | operator OR CRD
    - ALTHOUGH, it can be used | other Kubernetes resource
      - _Example:_ deployments, configMaps, ...

- == Argo CD plugin
  - allows
    - retrieve secrets -- from -- >=1 Secret Management tools
      - _Example of Secret Management tools:_ HashiCorp Vault, IBM Cloud Secrets Manager, AWS Secrets Manager
    - inject the secrets | Kubernetes resources

## Documentation

- [here](docs)

## Blogs
- [Solving ArgoCD Secret Management with the argocd-vault-plugin](https://itnext.io/argocd-secret-management-with-argocd-vault-plugin-539f104aff05)
- [Introducing argocd-vault-plugin v1.0!](https://itnext.io/introducing-argocd-vault-plugin-v1-0-708433294b2d)
- [How to Use HashiCorp Vault and Argo CD for GitOps on OpenShift](https://cloud.redhat.com/blog/how-to-use-hashicorp-vault-and-argo-cd-for-gitops-on-openshift)

## Presentations
- [Shh, It’s a Secret: Managing Your Secrets in a GitOps Way - Jake Wernette & Josh Kayani, IBM](https://youtu.be/7L6nSuKbC2c)
