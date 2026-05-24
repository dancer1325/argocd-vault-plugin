* goal
  * how to install "argocd-vault-plugin" | running cluster
    * ❌NO specify [configuration](/docs/config.md)❌

# requirements
* ⚠️if you want to use your Argo CD CLI -> 's version (MUST be) = [ArgoCD server installation](kustomization.yaml)⚠️
* run some secret management tool
  * _Example:_ [Vault](https://github.com/dancer1325/hashicorp-docs/blob/main/content/vault/v2.x/content/docs/deploy/kubernetes/helm/run.md)
    * create `kv/data/test` / key == `character`

# how to deploy it?
* requirements
  * [cluster running](https://github.com/dancer1325/argo-cd/blob/master/docs/examples/gettingStarted/README.md)
    * == `kubectl create namespace argocd`
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
    * ⚠️it takes some time⚠️
  * `kubectl exec -n argocd $(kubectl get pod -l app.kubernetes.io/name=argocd-repo-server -n argocd -o name) -c avp -- argocd-vault-plugin version`
    * check `argocd-vault-plugin` is running | "argocd-repo-server"
* test with an Application
  * `kubectl apply -f avp-backend-config.yaml`
    * creates the Secret | argocd namespace / AVP knows how to connect to Vault
    * ⚠️adjust `VAULT_TOKEN` & `VAULT_ADDR` to your environment⚠️
    * ⚠️replace `<your-vault-token>` with your actual token BEFORE applying⚠️
  * `kubectl apply -f example-app.yaml -n argocd`
  * check it works
    * `kubectl get application avp-cmp-sidecar-test -n argocd`
      * status should be `Synced` & `Healthy`
    * `kubectl get secret avp-test-secret -n default`
      * secret created with the value from Vault replacing the `<character>` placeholder


