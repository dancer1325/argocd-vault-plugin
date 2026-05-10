## how to install it?

* 💡[ways](https://github.com/dancer1325/argo-cd/blob/master/docs/operator-manual/config-management-plugins.md#how-to-install-a-config-management-plugin)💡
  * [-- by -- modifying "argocd-cm" ConfigMap](#---by----modifying-argocd-cm-configmap)
  * -- via a -- [custom plugins](https://argo-cd.readthedocs.io/en/stable/user-guide/config-management-plugins)

### -- by -- modifying "argocd-cm" ConfigMap

* | ArgoCD v2.4,
  * deprecated
* ⚠️| ArgoCD v2.8,
  * removed ⚠️

#### built-in image + initContainer

* steps
  * | "argocd-repo-server" manifest,
    * download the plugin -- as -- initcontainer
    * mount the plugin -- as -- volume
  * | "argocd-cm"'s `data.configManagementPlugins`,
    * register the plugin

#### custom image

* steps
  * build your custom image
  * | "argocd-repo-server" manifest,
    * set `spec.template.spec.containers[0].image: <CUSTOM_IMAGE>`
  * | "argocd-cm"'s `data.configManagementPlugins`,
    * register the plugin

### -- as -- sidecar plugin to repo-server

* ⚠️requirements⚠️
  * Argo CD v2.4.0

#### built-in image + initContainer

* [steps](https://github.com/dancer1325/argo-cd/blob/master/docs/operator-manual/config-management-plugins.md#---as----sidecar-plugin-to-repo-server)

#### custom Image

* [steps](https://github.com/dancer1325/argo-cd/blob/master/docs/operator-manual/config-management-plugins.md#---as----sidecar-plugin-to-repo-server)

## how to install locally?
### | Linux, OR macOS, -- via -- Curl
```
curl -Lo argocd-vault-plugin https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/{version}/argocd-vault-plugin_{version}_{linux|darwin}_{amd64|arm64|s390x}

chmod +x argocd-vault-plugin

mv argocd-vault-plugin /usr/local/bin
```

### | macOS, -- via -- Homebrew

```
brew install argocd-vault-plugin
```

## Security considerations

* ⚠️if somebody has access to the Argo CD repo-server OR Redis instance -> can access to these Kubernetes manifests + contained secrets⚠️ 
  * Reason of 
    * repo-server: 🧠secrets are injected | Kubernetes manifests / generated | Argo CD repo-server🧠
    * Redis:🧠these Kubernetes manifests + contained secrets are cached | Redis instance / used by Argo CD🧠
  * SOLUTION:👀
    * [mitigations](https://github.com/dancer1325/argo-cd/blob/master/docs/operator-manual/secret-management.md#how-to-mitigate-the-risks)
    * [provide OWN Redis credentials](https://github.com/dancer1325/argo-cd/blob/master/docs/faq.md#how-do-i-provide-my-own-redis-credentials)
