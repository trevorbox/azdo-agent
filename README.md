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
export AZP_TOKEN=mytoken
export AZP_URL=https://dev.azure.com/test
export AZP_POOL=default

helm upgrade -i secrets helm/secrets \
  --set azpdevops.url=${AZP_URL} \
  --set azpdevops.token=${AZP_TOKEN} \
  --set azpdevops.pool=${AZP_POOL} \
  -n azdo-agent


# this allows the service account to run the pod as privileged, since buildah needs root
oc adm policy add-scc-to-user privileged -z azdo-agent -n azdo-agent

# copy the builder serviceaccount's push secret which is used by the example /usr/bin/build.sh to test a build
oc get $(oc get secrets -n azdo-agent -o name | egrep builder-dockercfg) -n azdo-agent -o jsonpath={.data.\\.dockercfg} | base64 -d > /tmp/.dockercfg

oc create secret generic push-secret --from-file=.dockercfg=/tmp/.dockercfg --type="kubernetes.io/dockercfg"

helm upgrade -i azdo-agent helm/azdo-agent \
  --set image.repository=image-registry.openshift-image-registry.svc:5000/azdo-agent/azdo-agent \
  -n azdo-agent \
  --create-namespace
```

## test build in agent container

This will run the the example buildah script baked into the image to test an image build from the running agent.

```sh
oc exec deploy/azdo-agent -n azdo-agent -- /usr/bin/build.sh
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
