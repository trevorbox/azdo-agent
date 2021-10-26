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

# (optional) trigger a redeploy when the builder image changes
oc set triggers deploy/azdo-agent --from-image=azdo-agent:latest -c azdo-agent -n azdo-agent
```

## test build in agent container

This will run the the example buildah script baked into the image to test an image build from the running agent.

```sh
oc exec deploy/azdo-agent -n azdo-agent -- bash -c "time /usr/bin/build.sh"
```

## test improved performance from overlay cache

See <https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container#running_buildah_inside_a_container>

Spin the pod down and back up again. The pvc should have the /var/lib/contianers/storage cache and run faster 

```sh
oc rollout restart deploy/azdo-agent -n azdo-agent
```

example output of time to build...

```sh
...
real    0m42.198s
user    0m11.973s
sys     0m5.343s
```

After the pod is acaled up again, test the build to see faster time. Note: many pods using a RWO volume will wait to read/write and can cause unexpected behavior.

```sh
oc exec deploy/azdo-agent -n azdo-agent -- bash -c "time /usr/bin/build.sh"
```

example output of time to build...

```sh
...
real    0m15.238s
user    0m1.238s
sys     0m2.467s
```

This demonstrated a significant difference in time to build by using the cached image layers in storage pvc.

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
