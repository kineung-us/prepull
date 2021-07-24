# prepull with aks and acr (for now)

[![Build and Push](https://github.com/kineung-us/prepull/actions/workflows/build.yaml/badge.svg)](https://github.com/kineung-us/prepull/actions/workflows/build.yaml)
[![Release Helm](https://github.com/kineung-us/prepull/actions/workflows/chart.yaml/badge.svg)](https://github.com/kineung-us/prepull/actions/workflows/chart.yaml)
[![release](https://img.shields.io/github/v/release/kineung-us/prepull)](https://github.com/kineung-us/prepull/releases)

prepull is a tool that downloads the images you want to use to each node in Kubernetes. You can also use the dockerconfigjson file for a pull of images in your private repositories.

I am using [crictl][crictl] to work in recent Kubernetes which uses containerd engine, [jq][jq] for parsing dockerconfigjson. There is using the [pause image][pause] to make sure the whole run was done safely.

When the daemonset is installed, the specified image pull starts, and after completion, the helm chart is removed with the helm hook(post-install).

## How to use

Need to helm3.

Set values file like below.

```
## values.yaml

image:
  target:
    registry: "ghcr.io"
    repository: "kineung-us/nydus"
    tag: "0.1.4"

rbac:
  setup: true
```

Set chart and run install command.

```
helm repo add prepull https://kineung-us.github.io/prepull/
helm repo update

helm install prepull-nydus prepull --values values.yaml
```
## rbac

`.rbac.setup=true` is need only at fisrt time. Default value is `false`.

```
## values.yaml

image:
  target:
    registry: "ghcr.io"
    repository: "kineung-us/nydus"
    tag: "0.1.4"

rbac:
  setup: true
```

## With private repository

### setup secret

Here's the [howto][howto] Setup dockerconfigjson to secret.

If you set secret name `ghcr-auth`, you can print data.
```
kubectl get secret ghcr-auth --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
## output
## {"auths":{"your.private.registry.example.com":{"username":"janedoe","password":"xxxxxxxxxxx","email":"jdoe@example.com","auth":"c3R...zE2"}}}
```
* Need to check output have **username** and **password**.
* Need to check `.image.target.registry` and `"your.private.registry.example.com"` are same.

### setup values

Add `.auth.setup=true` and `.auth.dockerconfigjson="ghcr-auth"`.

```
## values-with-secret.yaml

image:
  target:
    registry: "ghcr.io"
    repository: "kineung-us/nydus"
    tag: "0.1.4"

auth:
  setup: true
  dockerconfigjson: "ghcr-auth"
```

## Maintainers

[@mrchypark](https://github.com/mrchypark).

## Contributing

Feel free to dive in! [Open an issue](https://github.com/kineung-us/prepull/issues/new) or submit PRs.

Standard Readme follows the [Contributor Covenant](https://www.contributor-covenant.org/version/2/0/code_of_conduct/) Code of Conduct.

### Contributors

This project exists thanks to all the people who contribute. 

## License

[MIT](LICENSE) © Chanyub Park

[jq]: https://stedolan.github.io/jq/
[crictl]: https://github.com/kubernetes-sigs/cri-tools
[pause]: https://github.com/kubernetes/kubernetes/tree/master/build/pause
[howto]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/