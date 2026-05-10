# TODO:

# how to install it?
## -- by -- modifying "argocd-cm" ConfigMap
### built-in image + initContainer
* [here -- as -- Kustomize app](/manifests/cmp-configmap)
### custom Image
* [here](asArgoCDCMViaCustomImageDockerfile)
TODO: install
## -- as -- sidecar plugin to repo-server
### built-in image + initContainer
* [here -- as -- Kustomize app](/manifests/cmp-sidecar)
### custom Image
* [configMap / nest `ConfigManagementPlugin`](asSideCarCMViaCustomImage.yaml)
* [custom image](asSideCarViaCustomImageDockerfile)
* patch "argocd-repo-server" deployment

# Security considerations
## ⚠️if somebody has access to the Argo CD repo-server OR Redis instance -> can access to these Kubernetes manifests + contained secrets⚠️
TODO: 
