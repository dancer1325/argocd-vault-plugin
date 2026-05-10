# TODO:
TODO:

# secret management | GitOps & Argo CD
## == 💡Argo CD Config Management Plugin💡
* [manifest / define the plugin itself](/manifests/cmp-sidecar/cmp-plugin.yaml)
### ❌NOT rely on | operator OR CRD❌
* there is NO
  * reconciliation loop | this repo
    * _Example:_ NOTHING found by "Reconcile()", "Watch()", "ctrl.Manager"
  * "controller-runtime" | [go.mod](/go.mod)
  * defined CRD

# allows
## retrieve secrets -- from -- >=1 Secret Management tools
* supported [backends](/pkg/backends)
## inject the secrets | Kubernetes resources
* | 
  * [deployment](/fixtures/input/nonempty/deployment.yaml)
  * [other resources](/pkg/kube/template_test.go)
