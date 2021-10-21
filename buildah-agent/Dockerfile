FROM registry.access.redhat.com/ubi8/buildah

ARG TARGETARCH=amd64
ARG AGENT_VERSION=2.185.1

WORKDIR /azp

COPY azure-cli.repo /etc/yum.repos.d/azure-cli.repo

COPY ./start.sh .

RUN if [ "$TARGETARCH" = "amd64" ]; then \
      AZP_AGENTPACKAGE_URL=https://vstsagentpackage.azureedge.net/agent/${AGENT_VERSION}/vsts-agent-linux-x64-${AGENT_VERSION}.tar.gz; \
    else \
      AZP_AGENTPACKAGE_URL=https://vstsagentpackage.azureedge.net/agent/${AGENT_VERSION}/vsts-agent-linux-${TARGETARCH}-${AGENT_VERSION}.tar.gz; \
    fi; \
    curl -LsS "$AZP_AGENTPACKAGE_URL" | tar -xz && \
    rpm --import https://packages.microsoft.com/keys/microsoft.asc && \
    bin/installdependencies.sh && \
    dnf install -y azure-cli git && \
    dnf clean all && \
    rm -rf /var/cache/dnf && \
    chown -R 1001:0 /azp &&  \
    chmod -R g+rwX /azp && \
    chmod +x start.sh

USER 1001

ENTRYPOINT [ "./start.sh" ]