FROM registry.access.redhat.com/ubi8/buildah


RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc

COPY azure-cli.repo /etc/yum.repos.d/azure-cli.repo

RUN dnf install -y azure-cli \
    && dnf clean all \
    && rm -rf /var/cache/yum

# RUN curl -LsS https://aka.ms/InstallAzureCLIDeb | bash \
#   && rm -rf /var/lib/apt/lists/*

ARG TARGETARCH=amd64
ARG AGENT_VERSION=2.185.1

WORKDIR /azp
RUN if [ "$TARGETARCH" = "amd64" ]; then \
      AZP_AGENTPACKAGE_URL=https://vstsagentpackage.azureedge.net/agent/${AGENT_VERSION}/vsts-agent-linux-x64-${AGENT_VERSION}.tar.gz; \
    else \
      AZP_AGENTPACKAGE_URL=https://vstsagentpackage.azureedge.net/agent/${AGENT_VERSION}/vsts-agent-linux-${TARGETARCH}-${AGENT_VERSION}.tar.gz; \
    fi; \
    curl -LsS "$AZP_AGENTPACKAGE_URL" | tar -xz


COPY ./start.sh .

RUN chown -R 1001:0 /azp &&  \
    chmod -R g+rwX /azp && \
    chmod +x start.sh

USER 1001

ENTRYPOINT [ "./start.sh" ]