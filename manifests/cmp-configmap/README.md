# how to deploy it?
* requirements
  * [cluster running](https://github.com/dancer1325/argo-cd/blob/master/docs/examples/gettingStarted/README.md)
    * ❌NO install ArgoCD❌
      * Reason:🧠
        * installed ALSO -- via -- kustomize
        * OTHERWISE, it fails🧠
* `kustomize build . | kubectl apply -n argocd -f -`
* check ALL was installed PROPERLY
  * `kubectl get configmap cmp-plugin -n argocd`
    * ConfigMap was installed
  * `kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-repo-server`
    * ALL "argocd-repo-server" containers are up
  * `kubectl exec -n argocd $(kubectl get pod -l app.kubernetes.io/name=argocd-repo-server -n argocd -o name) -c avp -- argocd-vault-plugin version`
    * check `argocd-vault-plugin` is running | "argocd-repo-server"
* TODO: add some Application / check its functionality


