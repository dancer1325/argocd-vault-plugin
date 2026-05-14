# TODO:
TODO:

# configure argocd-vault-plugin 
## -- via -- Kubernetes Secret
### requirements: "argocd-repo-server" has a service account token / mounted | standard location
TODO:
### steps
#### create secret / you can specify it | `data` OR `stringData`
* [here](k8sSecrets.yaml)
* `kubectl apply -f k8sSecrets.yaml`
#### `argocd-vault-plugin generate /some/path -s vault-configuration`
TODO:

# TODO:
TODO: