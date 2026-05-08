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

* [here](docs/blog)

## Presentations
- [Shh, It’s a Secret: Managing Your Secrets in a GitOps Way - Jake Wernette & Josh Kayani, IBM](https://youtu.be/7L6nSuKbC2c)
