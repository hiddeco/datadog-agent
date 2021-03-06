########################################
# Preparation stage: layout and chmods #
########################################

FROM debian:buster-slim as builder

WORKDIR /output

COPY datadog-cluster-agent opt/datadog-agent/bin/datadog-cluster-agent
COPY ./conf.d etc/datadog-agent/conf.d
COPY ./datadog-cluster.yaml etc/datadog-agent/datadog-cluster.yaml
COPY ./dist opt/datadog-agent/bin/dist
COPY entrypoint.sh .

RUN chmod 755 entrypoint.sh \
    && chmod g+r,g+w,g+X -R etc/datadog-agent/ \
    && chmod +x opt/datadog-agent/bin/datadog-cluster-agent \
    && ln -s /opt/datadog-agent/bin/datadog-cluster-agent opt/datadog-agent/bin/agent

####################################
# Actual docker image construction #
####################################

FROM debian:buster-slim

LABEL maintainer "Datadog <package@datadoghq.com>"

ENV PATH="/opt/datadog-agent/bin/:$PATH"

RUN apt-get update \
 && apt-get install --no-install-recommends -y ca-certificates curl \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder /output /

# Allow running as an unprivileged user:
# - General case is the dd-agent user
# - OpenShift uses a random UID in the root group
#
# Containerd does not preserve permissions when mounting a volume on top
# of an empty folder. Creating .placeholder files as a workaround.
#
RUN adduser --system --no-create-home --disabled-password --ingroup root dd-agent \
 && mkdir -p /var/log/datadog/ \
 && touch /var/log/datadog/.placeholder \
 && touch /tmp/.placeholder \
 && chown -R dd-agent:root /etc/datadog-agent/ /var/log/datadog/ /tmp/ \
 && chmod g+r,g+w,g+X -R /etc/datadog-agent/ /var/log/datadog/ /tmp/

USER dd-agent

# Leave following directories RW to allow use of readonly rootfs
VOLUME ["/etc/datadog-agent", "/var/log/datadog", "/tmp"]

ENTRYPOINT ["/entrypoint.sh"]

# No docker healthcheck, use a HTTP check
# on port 5005 and/or 443 on Kubernetes

CMD ["datadog-cluster-agent", "start"]
