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
oc new-build https://github.com/trevorbox/azdo-agent.git --strategy=docker --context-dir=agent/ -n azdo-agent
```

# deploy

```sh
export AZP_TOKEN=${PAT}
export AZP_URL=${URL}
export AZP_POOL=${POOL}

helm upgrade -i azdo-agent helm/azdo-agent \
  --set image.repository=quay.io/trevorbox/azdo-agent \
  --set azpdevops.token=${AZP_TOKEN} \
  --set azpdevops.url=${AZP_URL} \
  --set azpdevops.pool=${AZP_POOL} \
  -n azdo-agent \
  --create-namespace
```

# test local

```sh
podman build -t test .
podman run -it --entrypoint="/bin/bash" -e AZP_TOKEN=$PAT -e AZP_URL=$URL --user 1001 test
podman run -it -e AZP_TOKEN=$PAT -e AZP_URL=$URL --user 1001 test
```