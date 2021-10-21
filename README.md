# azdo-agent

This follows the steps noted in <https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops> but with permissions for Openshift.

# build/push locally

```sh
podman build -t quay.io/trevorbox/azdo-agent:latest -f agent/Dockerfile
podman login quay.io
podman push quay.io/trevorbox/azdo-agent:latest
```

# (optional) build ocp

```sh
oc new-project azdo-agent
oc new-build https://github.com/trevorbox/azdo-agent.git --strategy=docker --context-dir=buildah-agent/ -n azdo-agent
```

# deploy

```sh
export AZP_TOKEN=
export AZP_URL=
export AZP_POOL=

helm upgrade -i azdo-agent helm/azdo-agent \
  --set image.repository=image-registry.openshift-image-registry.svc:5000/azdo-agent/azdo-agent \
  --set azpdevops.token=${AZP_TOKEN} \
  --set azpdevops.url=${AZP_URL} \
  --set azpdevops.pool=${AZP_POOL} \
  -n azdo-agent \
  --create-namespace
```

# test local

buildah-agent

```sh
podman build -t buildah-agent -f buildah-agent/Dockerfile
podman run -it --entrypoint="/bin/bash" -e AZP_TOKEN=${AZP_TOKEN} -e AZP_URL=${AZP_URL} --user 1001 buildah-agent
podman run -it -e AZP_TOKEN=${AZP_TOKEN} -e AZP_URL=${AZP_URL} --user 1001 buildah-agent
```

agent

```sh
podman build -t agent -f agent/Dockerfile
podman run -it --entrypoint="/bin/bash" -e AZP_TOKEN=${AZP_TOKEN} -e AZP_URL=${AZP_URL} --user 1001 agent
podman run -it -e AZP_TOKEN=${AZP_TOKEN} -e AZP_URL=${AZP_URL} --user 1001 agent
```