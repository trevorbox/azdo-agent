# ubi9 base image

> <https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#linux>

```sh
export AZP_URL=""
export AZP_TOKEN=""
export AZP_POOL=""

export TAG="quay.io/trevorbox/azp-agent-ubi9:1.0.1" # replace with your tag

podman build -t $TAG . \
  --secret=id=azp_url,env=AZP_URL \
  --secret=id=azp_token,env=AZP_TOKEN \
  --build-arg git_origin_url=$(git config --get remote.origin.url) \
  --build-arg git_revision=$(git rev-parse HEAD) \
  --build-arg base_image_digest=$(skopeo inspect --format "{{ .Digest }}" docker://registry.access.redhat.com/ubi9/ubi:latest) \
  --build-arg base_image_repository=registry.access.redhat.com/ubi9/ubi \
  --build-arg base_image_tag=latest \
  --build-arg src_version=$(git rev-parse --abbrev-ref HEAD) \
  --build-arg created=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg author_emails="myorg@example.com" \
  --build-arg build_host=$(uname -n) \
  --build-arg build_id="1"

podman login quay.io
podman push $TAG

podman run -it --rm -e AZP_URL="$AZP_URL" -e AZP_TOKEN="$AZP_TOKEN" -e AZP_POOL="$AZP_POOL" -e AZP_AGENT_NAME="Docker Agent - Linux" --name "azp-agent-linux" $TAG
podman run -it --rm --entrypoint=/bin/bash -e AZP_URL="$AZP_URL" -e AZP_TOKEN="$AZP_TOKEN" -e AZP_POOL="$AZP_POOL" -e AZP_AGENT_NAME="Docker Agent - Linux" --name "azp-agent-linux" $TAG

podman run -it --rm --entrypoint=/bin/bash --name "azp-agent-linux" $TAG
```

## Gator

```sh
export TAG="quay.io/trevorbox/azp-agent-ubi9-gator:0.0.1"
podman build -f Containerfile-gator -t $TAG
```

## Deploy

```sh
export AZP_URL=""
export AZP_TOKEN=""
export AZP_POOL=""
export VSTS_HTTP_PROXY=""
export VSTS_HTTP_NO_PROXY=""
export VSTS_HTTP_PROXY_USERNAME=""
export VSTS_HTTP_PROXY_PASSWORD=""

helm upgrade -i azp-agent-secret --create-namespace -n azp-agent ../helm/secrets --set azpdevops.url="$AZP_URL" --set azpdevops.token="$AZP_TOKEN" --set azpdevops.pool="$AZP_POOL" \
--set azpdevops.proxy="$AZP_POOL" \
--set azpdevops.no_proxy="$AZP_POOL" \
--set azpdevops.proxy_username="$AZP_POOL" \
--set azpdevops.proxy_password="$AZP_POOL" \
helm upgrade -i azp-agent --create-namespace -n azp-agent ../helm/azp-agent 
```