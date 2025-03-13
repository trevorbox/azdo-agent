# ubi9 base image

```sh
export TAG="quay.io/trevorbox/azp-agent:ubi9" # replace with your tag

podman build -t $TAG . \
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

podman build -t quay.io/trevorbox/azp-agent:ubi9 .
podman login quay.io
podman push quay.io/trevorbox/azp-agent:ubi9
podman run -it --rm -e AZP_URL="<Azure DevOps instance>" -e AZP_TOKEN="<Personal Access Token>" -e AZP_POOL="<Agent Pool Name>" -e AZP_AGENT_NAME="Docker Agent - Linux" --name "azp-agent-linux" quay.io/trevorbox/azp-agent:ubi9

podman run --rm -it --entrypoint=sh quay.io/trevorbox/azp-agent:ubi9
```
