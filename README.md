# azdo-agent

```sh
oc new-project azdo-agent
oc new-build https://github.com/trevorbox/azdo-agent.git --strategy=docker --context-dir=agent/ -n azdo-agent

export AZP_TOKEN=$PAT
export AZP_URL=$URL

oc create secret generic azdo-agent --from-literal=AZP_TOKEN=$AZP_TOKEN --from-literal=AZP_URL=$AZP_URL -n azdo-agent

helm upgrade -i azdo-agent helm/azdo-agent -n azdo-agent

```

# local

```sh
podman build -t test .
podman run -it --entrypoint="/bin/bash" -e AZP_TOKEN=$PAT -e AZP_URL=$URL --user 1001 test
podman run -it -e AZP_TOKEN=$PAT -e AZP_URL=$URL --user 1001 test
```