## how to install Argo CD?

* ways
  * -- via -- "argocd-cm" ConfigMap
    * download AVP | volume + control ALL -- as -- Kubernetes manifests
      * [pre-built Kustomize app](/manifests/cmp-configmap)
    * create a custom "argocd-repo-server" image / 
      * contains AVP 
      * support tools pre-installed
  * -- via a -- [custom plugins](https://argo-cd.readthedocs.io/en/stable/user-guide/config-management-plugins)
    * ⚠️requirements⚠️
      * Argo CD v2.4.0
    * == [define sidecar container / EACH individual plugin](https://argo-cd.readthedocs.io/en/stable/user-guide/config-management-plugins/#installing-a-cmp)
      * | the plugin runs
    * download AVP & supporting tools | volume + control ALL -- as -- Kubernetes manifests
      * [pre-built Kustomize app](/manifests/cmp-sidecar)
    * create a custom sidecar image /
      * contains AVP
      * support tools pre-installed

### Explaining your options

* ways to extend the "argocd-repo-server"
  * ADDITIONAL tools
  * [custom built image](https://argoproj.github.io/argo-cd/operator-manual/custom_tools/)

TODO: 
There are some [security benefits to running this way](https://github.com/argoproj/argo-cd/issues/9083#issuecomment-1098517762), it may be [future proof](https://github.com/argoproj/argo-cd/issues/8117), and 
you don't have to explicitly tell Argo CD which plugin to use: it will auto-detect it,
like it does for Helm or Kustomize based applications. On the other hand, 
it adds a bit more complexity and can make some argocd-vault-plugin integrations a bit trickier 
see the [caveats section of the Usage page](../usage#running-argocd-vault-plugin-in-a-sidecar-container) for details.

### InitContainer and configuration via argocd-cm ConfigMap

The first technique is to use an init container and a volumeMount to copy a different version of a tool into the repo-server container.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-repo-server
spec:
  template:
    spec:
      containers:
      - name: argocd-repo-server
        volumeMounts:
        - name: custom-tools
          mountPath: /usr/local/bin/argocd-vault-plugin
          subPath: argocd-vault-plugin

        # Note: AVP config (for the secret manager, etc) can be passed in several ways. This is just one example
        # https://argocd-vault-plugin.readthedocs.io/en/stable/config/
        envFrom:
          - secretRef:
              name: argocd-vault-plugin-credentials
      volumes:
      - name: custom-tools
        emptyDir: {}
      initContainers:
      - name: download-tools
        image: alpine:3.8
        command: [sh, -c]

        # Don't forget to update this to whatever the stable release version is
        # Note the lack of the `v` prefix unlike the git tag
        env:
          - name: AVP_VERSION
            value: "1.18.0"
        args:
          - >-
            wget -O argocd-vault-plugin
            https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${AVP_VERSION}/argocd-vault-plugin_${AVP_VERSION}_linux_amd64 &&
            chmod +x argocd-vault-plugin &&
            mv argocd-vault-plugin /custom-tools/
        volumeMounts:
          - mountPath: /custom-tools
            name: custom-tools

      # Not strictly necessary, but required for passing AVP configuration from a secret and for using Kubernetes auth to Hashicorp Vault
      automountServiceAccountToken: true
```

### Custom Image and configuration via argocd-cm ConfigMap

The following example builds an entirely customized repo-server from a Dockerfile, installing extra dependencies that may be needed for generating manifests.

```Dockerfile
FROM argoproj/argocd:latest

# Switch to root for the ability to perform install
USER root

# Install tools needed for your repo-server to retrieve & decrypt secrets, render manifests
# (e.g. curl, awscli, gpg, sops)
RUN apt-get update && \
    apt-get install -y \
        curl \
        awscli \
        gpg && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install the AVP plugin (as root so we can copy to /usr/local/bin)
ENV AVP_VERSION=1.18.0
ENV BIN=argocd-vault-plugin
RUN curl -L -o ${BIN} https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${AVP_VERSION}/argocd-vault-plugin_${AVP_VERSION}_linux_amd64
RUN chmod +x ${BIN}
RUN mv ${BIN} /usr/local/bin

# Switch back to non-root user
USER 999
```
After making the plugin available, you must then register the plugin, documentation can be found at <https://argoproj.github.io/argo-cd/user-guide/config-management-plugins/#plugins> on how to do that.

For this plugin, you would add this:
```yaml
data:
  configManagementPlugins: |-
    - name: argocd-vault-plugin
      generate:
        command: ["argocd-vault-plugin"]
        args: ["generate", "./"]
```

You can use ArgoCD Vault Plugin along with other Kubernetes configuration tools (Helm, Kustomize, etc)
* The general method is to have your configuration tool output YAMLs that are ready to apply to a cluster except for containing `<placeholder>`s, and then run the plugin on this output to fill in the secrets
* See the [Usage page](../usage) for examples.

### InitContainer and configuration via sidecar

Define the plugin in a ConfigMap that will be mounted in the sidecar container
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cmp-plugin
data:
  avp.yaml: |
    apiVersion: argoproj.io/v1alpha1
    kind: ConfigManagementPlugin
    metadata:
      name: argocd-vault-plugin
    spec:
      allowConcurrency: true
      discover:
        find:
          command:
            - sh
            - "-c"
            - "find . -name '*.yaml' | xargs -I {} grep \"<path\\|avp\\.kubernetes\\.io\" {} | grep ."
      generate:
        command:
          - argocd-vault-plugin
          - generate
          - "."
      lockRepo: false
---
```

Patch the argocd-repo-server to add an initContainer to download argocd-vault-plugin and define the sidecar
* You can change the image from `registry.access.redhat.com/ubi8` to whatever is desired, so long as it [contains the needed binaries](../usage#running-argocd-vault-plugin-in-a-sidecar-container)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-repo-server
spec:
  template:
    spec:
      automountServiceAccountToken: true
      volumes:
        - configMap:
            name: cmp-plugin
          name: cmp-plugin
        - name: custom-tools
          emptyDir: {}
      initContainers:
      - name: download-tools
        image: registry.access.redhat.com/ubi8
        env:
          - name: AVP_VERSION
            value: 1.18.0
        command: [sh, -c]
        args:
          - >-
            curl -L https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v$(AVP_VERSION)/argocd-vault-plugin_$(AVP_VERSION)_linux_amd64 -o argocd-vault-plugin &&
            chmod +x argocd-vault-plugin &&
            mv argocd-vault-plugin /custom-tools/
        volumeMounts:
          - mountPath: /custom-tools
            name: custom-tools
      containers:
      - name: avp
        command: [/var/run/argocd/argocd-cmp-server]
        image: registry.access.redhat.com/ubi8
        securityContext:
          runAsNonRoot: true
          runAsUser: 999
        volumeMounts:
          - mountPath: /var/run/argocd
            name: var-files
          - mountPath: /home/argocd/cmp-server/plugins
            name: plugins
          - mountPath: /tmp
            name: tmp
          
          # Register plugins into sidecar
          - mountPath: /home/argocd/cmp-server/config/plugin.yaml
            subPath: avp.yaml
            name: cmp-plugin

          # Important: Mount tools into $PATH
          - name: custom-tools
            subPath: argocd-vault-plugin
            mountPath: /usr/local/bin/argocd-vault-plugin
```

### Custom Image and configuration via sidecar
Define the plugin in a ConfigMap that will be mounted in the sidecar container
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cmp-plugin
data:
  avp.yaml: |
    apiVersion: argoproj.io/v1alpha1
    kind: ConfigManagementPlugin
    metadata:
      name: argocd-vault-plugin
    spec:
      allowConcurrency: true
      discover:
        find:
          command:
            - sh
            - "-c"
            - "find . -name '*.yaml' | xargs -I {} grep \"<path\\|avp\\.kubernetes\\.io\" {} | grep ."
      generate:
        command:
          - argocd-vault-plugin
          - generate
          - "."
      lockRepo: false
---
```

Define a sidecar image from a suitable base
```Dockerfile
FROM registry.access.redhat.com/ubi8

# Switch to root for the ability to perform install
USER root

# Install tools needed for your repo-server to retrieve & decrypt secrets, render manifests
# (e.g. curl, awscli, gpg, sops)
RUN apt-get update && \
    apt-get install -y \
        curl \
        awscli \
        gpg && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install the AVP plugin (as root so we can copy to /usr/local/bin)
ENV AVP_VERSION=1.18.0
ENV BIN=argocd-vault-plugin
RUN curl -L -o ${BIN} https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${AVP_VERSION}/argocd-vault-plugin_${AVP_VERSION}_linux_amd64
RUN chmod +x ${BIN}
RUN mv ${BIN} /usr/local/bin

# Switch back to non-root user
USER 999
```

Patch the argocd-repo-server to define the sidecar
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-repo-server
spec:
  template:
    spec:
      automountServiceAccountToken: true
      volumes:
        - configMap:
            name: cmp-plugin
          name: cmp-plugin
      containers:
      - name: avp
        command: [/var/run/argocd/argocd-cmp-server]
        image: your-container-registry/your-custom-image
        securityContext:
          runAsNonRoot: true
          runAsUser: 999
        volumeMounts:
          - mountPath: /var/run/argocd
            name: var-files
          - mountPath: /home/argocd/cmp-server/plugins
            name: plugins
          - mountPath: /tmp
            name: tmp
          
          # Register plugins into sidecar
          - mountPath: /home/argocd/cmp-server/config/plugin.yaml
            subPath: avp.yaml
            name: cmp-plugin
```

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

* Argo CD Vault Plugin
  * injects secrets | Kubernetes manifests / generated | Argo CD repo-server
    * ⚠️if somebody has access to the Argo CD repo-server OR Redis instance -> can access to these Kubernetes manifests + contained secrets⚠️ 
      * Reason of Redis:🧠these Kubernetes manifests + contained secrets are cached | Redis instance / used by Argo CD🧠

* 👀mitigations👀
  1. set up network policies / prevent direct access -- to -- Argo CD components (Redis and the repo-server)
  2. run Argo CD | its OWN cluster (==standalone)
  3. [enable password authentication | Redis instance](https://github.com/argoproj/argo-cd/issues/3130)
